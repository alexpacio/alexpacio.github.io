.. title: RAM is expensive now: rewrite your backend in Rust
.. slug: rewrite-it-in-rust-ram-is-expensive
.. date: 2026-10-10 12:00:00 UTC+02:00
.. tags: rust, tokio, axum, nodejs, llms, ai, performance, sustainability, backend
.. category: programming
.. description: With RAM and storage getting pricier, efficiency is no longer a nice-to-have. AI-assisted Rust makes low-level backends accessible, and slop is the thing we should rewrite first.
.. type: text

For about fifteen years we told ourselves a comfortable story: hardware is cheap, developers are expensive, so write the slowest thing that ships and scale it horizontally. Node, Python, Ruby, PHP, a sprinkle of JVM. Add another replica. Bump the instance size. Move on.

That story had a hidden assumption: that RAM and storage would keep getting cheaper, forever, so nobody would ever have to look at the bill. That assumption is now broken. DRAM and NAND prices have been climbing hard as AI datacenters absorb a huge share of the supply, and nobody serious expects the cost of memory-hungry infrastructure to go back to "who cares" any time soon.

When memory is the expensive resource, the language you pick for your backend stops being a matter of taste. It becomes a line item.

The cost you were not looking at
================================

A managed-runtime backend carries its runtime with it: the VM, the JIT, the garbage collector and its headroom, the framework, the ORM, the validation library and a few hundred transitive dependencies. Multiply that by replicas, by environments (prod, staging, preview), by regions, and by the sidecars and node overhead that come with the container orchestration you wrapped it in.

A service written with Axum on Tokio is a single native binary with no runtime to pay for. Let's look at what that means in numbers, for the stacks people actually run in production: Java with Spring Boot, C# with ASP.NET Core, and Rust with Axum.

Numbers, not vibes
==================

**Memory.** The most useful public data I found is `Sharkbench <https://sharkbench.dev/web>`__, an open-source (Apache-2.0) benchmark that runs the same web workload against every framework: concurrent HTTP requests, I/O and JSON (de)serialization, on Linux with an Intel Core Ultra 5 325, each app in a Docker container limited to one core-equivalent (see the `repository README <https://github.com/sharkbench/sharkbench>`__). These are the figures as of 2026-10-05:

.. list-table::
   :header-rows: 1
   :widths: 34 16 14 16 20

   * - Stack
     - Req/s
     - Latency
     - Memory
     - vs Axum
   * - Rust, Axum 0.8.9
     - 26,909
     - 1.2 ms
     - **5.7 MB**
     - 1x
   * - C#, ASP.NET Core (.NET 10)
     - 17,561
     - 1.5 ms
     - 46.6 MB
     - ~8x memory
   * - JavaScript, Express 5 (Node.js)
     - 7,670
     - 3.7 ms
     - 84.8 MB
     - ~15x memory
   * - Java, Spring Boot 4.1 WebFlux (Temurin 25)
     - 4,519
     - 1.7 ms
     - 237.0 MB
     - ~42x memory
   * - Java, Spring Boot 4.1 MVC (Temurin 25)
     - 3,316
     - 1.6 ms
     - 244.5 MB
     - ~43x memory

(Sources: the `Axum <https://sharkbench.dev/web/rust-axum>`__, `ASP.NET Core <https://sharkbench.dev/web/csharp-aspnetcore>`__ and `Spring Boot <https://sharkbench.dev/web/java-springboot>`__ result pages.)

Read it fairly, because it's not a pure win for one side:

* The benchmark doesn't document whether memory is peak or average, or whether the container had a memory cap. Without a cap the JVM sizes its heap from the machine's RAM, so a good chunk of those 240 MB is the JVM taking what it's offered. The same Spring Boot on the IBM Semeru runtime lands around 130 MB in the same benchmark, and a tuned heap can go lower still. That is exactly the point, though: with Rust, you don't have to tune the problem away.
* Java can be extremely fast: Vert.x on Temurin tops the throughput chart at 34,188 req/s, ahead of Axum, but at 268.8 MB of memory. Raw speed isn't the issue; the bill for the memory you need to get it is.
* It's a single benchmark with a simple workload, run with a one-core limit. A real service with a database driver, an ORM and a cache will weigh more on every stack.

**I also measured one end of this myself.** I built the minimal service from this post (Axum 0.8.9 + Tokio + serde, one JSON endpoint, Rust 1.98.1, release build with LTO and ``strip``) and ran it on my machine. It uses **3.7 MB RSS idle**, and the peak was **6.5 MB after 320,000 requests** over 64 keep-alive connections. That agrees with the 5.7 MB Sharkbench reports, even though the load generator was a throwaway Python script, so treat it as indicative, not as a benchmark.

**Disk.** I couldn't find an authoritative, apples-to-apples public comparison for storage, so I went to the primary source and asked the registries directly (compressed sizes of the linux/amd64 images, queried on 2026-10-10):

.. list-table::
   :header-rows: 1
   :widths: 60 40

   * - What you ship
     - Size
   * - ``eclipse-temurin:25-jre`` (Java base image, before your fat JAR)
     - 120.1 MB
   * - ``mcr.microsoft.com/dotnet/aspnet:10.0`` (before your app)
     - 95.9 MB
   * - ``mcr.microsoft.com/dotnet/aspnet:10.0-alpine`` (before your app)
     - 54.3 MB
   * - The Axum service above: the whole stripped binary
     - 0.8 MB

The Axum service is the *application*; the other rows are just the platform it would run on. To be fair about that last row: the binary I built links dynamically against glibc, so to run it from ``scratch`` you'd build for the ``musl`` target (a little larger) or ship it on a small distroless base. Either way you land one to two orders of magnitude below a JRE or .NET runtime image before the first line of your own code, and in practice you pay that in registry storage, pull time on every deploy and cold start.

At small scale you can shrug this off. At any scale where the infrastructure invoice is a conversation in your company, you can't.

Why we didn't do it before
==========================

Honestly? Because it was hard, and the hardness was mostly *ergonomics*, not performance:

* the borrow checker felt like a hazing ritual,
* the async story was fragmented and the ecosystem was young,
* writing a CRUD endpoint took three times as long as in Express,
* finding people who knew the language was difficult.

Every one of those objections has aged badly.

The ecosystem is mature
=======================

You can build everything you build with Node today, with abstractions that are just as comfortable:

* **Tokio** is the async runtime. ``async``/``await`` works the way you expect, with a work-stealing scheduler and no event loop you can block by accident with one synchronous call.
* **Axum** gives you routing, extractors and middleware on top of Tower. It's what Express wanted to be if it had types.
* **serde** replaces the hand-written validation and (de)serialization glue; the type *is* the schema.
* **sqlx** gives you async database access with queries checked against your real schema at compile time.
* **tracing**, **reqwest**, **tower-http**, **clap**, **tokio-postgres**, and so on cover logging, HTTP clients, CORS, compression, CLIs...

An endpoint looks like this:

.. code:: rust

    use axum::{
        extract::{Path, State},
        http::StatusCode,
        routing::get,
        Json, Router,
    };
    use serde::Serialize;
    use sqlx::PgPool;

    #[derive(Serialize, sqlx::FromRow)]
    struct User {
        id: i64,
        name: String,
    }

    async fn get_user(
        State(pool): State<PgPool>,
        Path(id): Path<i64>,
    ) -> Result<Json<User>, StatusCode> {
        sqlx::query_as::<_, User>("SELECT id, name FROM users WHERE id = $1")
            .bind(id)
            .fetch_optional(&pool)
            .await
            .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?
            .map(Json)
            .ok_or(StatusCode::NOT_FOUND)
    }

    #[tokio::main]
    async fn main() {
        let pool = PgPool::connect(&std::env::var("DATABASE_URL").unwrap())
            .await
            .unwrap();

        let app = Router::new()
            .route("/users/{id}", get(get_user))
            .with_state(pool);

        let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
        axum::serve(listener, app).await.unwrap();
    }

(Trimmed for the blog: in production you'd map errors properly and handle startup failures without ``unwrap()``.)

That's not much more ceremony than the Express version, and you get a typed, compiled, memory-safe binary out of it.

The real shift: AI removed the learning cliff
=============================================

The remaining barrier was *you*: the time it takes to become fluent enough in a stricter language to stop fighting it. That's exactly the kind of barrier an LLM flattens.

A coding assistant is excellent at the parts of Rust that used to be the wall: lifetime errors, trait bounds, ``Send``/``Sync`` complaints on a future, choosing between ``Arc<Mutex<_>>`` and a channel. The compiler, which is strict and verbose, gives the model a precise feedback loop; you get a conversation with a tutor and a ruthless reviewer at the same time. And because Rust's errors are explicit, the model's mistakes tend to fail *at compile time*, not at 3 a.m. in production.

I'm not talking about vibe coding. I'm talking about the compiler and the model keeping each other honest while you stay the person who understands the design.

The paradox: AI can kill slop
=============================

The usual complaint is that AI floods the world with slop: huge, plausible, redundant JavaScript nobody reads. That's real. But the same tool can go the other way.

Ask an assistant to take that 40-file Node service, with its dependency pyramid, its three competing date libraries and its "temporary" helpers, and re-express its *behavior* in Rust. A rewrite used to be the classic managerial nightmare, months of work for zero features. Now the translation cost collapses, tests pin the behavior, and the output has to survive a compiler that rejects whole categories of sloppiness: no null surprises, no forgotten error paths, no data races, no ``undefined is not a function``.

Slop is cheap to produce in a language that tolerates it. It's much harder to produce in one that doesn't. If AI is going to write a lot of our code anyway, make it write in the language that pushes back.

Not a crusade (conditions apply)
================================

Let me be honest about where this doesn't apply:

* **Prototypes and throwaway scripts.** If it lives for a week, write it in whatever is fastest. Just be honest about when "temporary" expires.
* **Real technical constraints.** A library you depend on only exists in another ecosystem, a platform without a viable target, a team that can't take on the operational risk right now. These are legitimate. "I'd rather not learn it" isn't.
* **Workloads that aren't about your code.** If you're a thin layer over an ML runtime or a database doing the heavy lifting, your language isn't the bottleneck.
* **Compile times and iteration speed.** Rust builds are slower than ``node index.js``. Caching, workspaces and incremental builds help, but it's a real cost.
* **You still have to review.** Models invent crates and APIs that don't exist, and they'll happily reach for ``clone()`` everywhere to silence the borrow checker. A passing build is not the same as a good design.

But look at that list: it's all *operational and technical* reasons. The old reasons, "it's too hard", "nobody knows it", "the tooling isn't there", are gone.

I'm still against "everything in Rust"
=======================================

Let me be very clear, because this post can be read as a crusade and it isn't one. I don't believe in the "rewrite it all in Rust" philosophy, and in most cases rewriting working software makes no sense at all.

* **Don't rewrite what has years behind it.** Code that has been running in production for years and is battle tested carries something a rewrite can't: every bug already found, every edge case already handled, every weird client already accommodated. That knowledge lives in the code, not in the spec. A rewrite throws it away and makes you pay for it again, in production, with your users. A new, clean, fast codebase is not better than an old one that has stopped surprising you.
* **Don't rewrite C or C++ in Rust just to do it.** The argument of this post is about escaping the cost of a *managed runtime*: the VM, the JIT, the garbage collector, the interpreter. A service already written in C or C++ is already on the other side of that line. It has no runtime to pay for, and its memory footprint is already in the same ballpark as Rust's. (Drogon, the C++ framework, sits at 7.2 MB in the same Sharkbench table above, next to Axum's 5.7 MB.) Rewriting it saves you nothing on the bill you came here to reduce. Memory safety is a legitimate reason to choose Rust for *new* code, and a good reason to be careful around untrusted input, but it's not by itself a reason to burn years replacing something that works.
* **Don't pick Rust where another low-level language fits the job better.** Go is excellent for networking tools and small services, C remains the natural language of kernels, drivers and embedded targets, and C++ has ecosystems (game engines, high-performance numerical and graphics code, decades of existing libraries) where Rust is not the obvious choice. Pick the tool for the purpose, not the one with the best slogan.
* **A language is not a goal.** Nobody's customers care what the binary was written in. They care that it's fast, stable and cheap to run, and Rust is just one very good way to get there for a specific kind of problem: a backend that today runs on a heavy managed runtime and is going to be written or substantially rewritten anyway.

So the claim is a narrow one, and I'm happy to defend that narrow version: **when you are writing a new backend, or already have a good reason to rewrite one, and the alternative is a high-level managed runtime, there is much less reason than there used to be to avoid Rust.** That's all. It's not a mandate to touch anything else.

So what's the excuse now?
=========================

Every megabyte you waste is a megabyte somebody has to buy, power and cool, and at current prices you're now paying a premium for it. Choosing a heavyweight stack by default for new work, when a leaner one is within reach of any competent backend engineer with an assistant, isn't neutral. It's burning someone's money and electricity to save yourself a little discomfort. Those who keep doing it, absent a real constraint, are contributing to the waste.

I've written before about why I think `LLMs will push backend engineering toward low-level languages <link://slug/extreme-programming-llms-and-rust>`_ and about `what sustainable software looks like for small developers <link://slug/software-sustainability>`_. This is the same argument, with a price tag on it.

If you have a service that was already due for a rewrite, pick the one with the biggest memory footprint, or the one that costs the most to keep alive. Rewrite it in Rust with Tokio and Axum, with an assistant next to you and the old test suite as the spec. Look at the memory graph afterwards. If it's stable, battle tested and already lean, leave it alone.

Then decide whether you still need that excuse.
