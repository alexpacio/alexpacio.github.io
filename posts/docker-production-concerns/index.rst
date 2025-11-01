.. title: Containerization to run apps in production is about to die
.. slug: docker-production-concerns
.. date: 2025-11-01 12:00:00 UTC
.. tags: docker, containers, devops, production, infrastructure
.. category: Development
.. author: Alessandro Bolletta
.. summary: A critical look at Docker's role in production systems and why containerization might be adding unnecessary complexity to your infrastructure.

Containerization has become ubiquitous in modern software development, often presented as the default choice for deploying applications. However, after years of working with containerized systems in production, I've come to question whether **containers are truly the right solution** for production environments—or if we've collectively adopted a technology that excels at prototyping but introduces unnecessary complexity when it comes to running reliable, production-grade systems.

.. TEASER_END

Understanding my statement requires a deep knowledge of how linux containers actually work
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is a good document to read: https://www.redhat.com/en/topics/containers/whats-a-linux-container

Docker's Sweet Spot: Prototyping and Development
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let me be clear: Docker is **excellent** for certain use cases. When you need to quickly spin up an isolated environment, reproduce a bug, or share a development setup across a team, Docker shines. The ability to package dependencies, create reproducible environments, and isolate processes makes it invaluable for **prototyping** and **development workflows**.

Need to test a new database version? Spin up a container. Want to ensure your application works across different environments? Docker provides that consistency. For these scenarios, containerization offers genuine value with minimal overhead.

The problem begins when we assume that what works well for development automatically translates to production.

The Hidden Complexity of Production Containers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Running Docker in production introduces layers of complexity that are often underestimated. You're not just running your application anymore—you're managing an **entire orchestration layer**. Whether it's Docker Swarm, Kubernetes, or another container orchestration platform, you've added significant infrastructure overhead.

This complexity manifests in several ways: **networking becomes abstracted** through overlay networks, **storage requires volume management** and persistence strategies, **monitoring needs container-specific tooling**, and **debugging becomes harder** when you can't simply SSH into a traditional server and inspect processes directly.

Each layer of abstraction adds potential failure points. What was once a straightforward deployment becomes a complex dance of images, registries, orchestrators, and networking policies.

Wrapping a process into a virtual network host is a really BAD idea
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is one of the single greatest sources of complexity in containerized systems. On a traditional server, networking is simple: one host, one IP, one routing table. When you wrap your application in a container, you are giving it its own private network stack.

This abstraction creates a labyrinth. Instead of simple, predictable server-to-server communication, we now deal with **overlay networks**, **service meshes**, and **complex routing rules** just to get two services to talk. Debugging a connection issue is no longer a simple `netstat` or `tcpdump`; it's a deep investigation across multiple, virtualized layers.

Furthermore, this encapsulation is **inherently inefficient**. We are adding network hops and processing overhead for every single packet, slowing down communication for the sake of an isolation boundary that is often unnecessary.

Having foreign applications running into the same OS is also a really BAD, insecure idea
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's be blunt: **containers are not VMs**. They all share the same host kernel.

While namespaces and cgroups provide a degree of isolation, they are not a true security boundary. A single kernel-level vulnerability can (and has) led to container escapes, allowing a process in one "isolated" container to break out and gain control of the host—and by extension, *all other containers* running on it.

Running multiple "foreign" applications, potentially from different teams or with different security requirements, on the same OS kernel is a ticking time bomb. We are trading robust, hardware-enforced isolation (like that from virtual machines) for a flimsy, software-based partition.

Having your own applications running into the machines of your cloud provider, where the provider can actually access your data, is a really BAD and DRAMATICALLY insecure idea
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This problem is magnified in the cloud. When you use a managed Kubernetes service (like EKS, GKE, or AKS), you are running your applications on a host OS that you don't control, managed by a provider whose main business is multi-tenancy.

You are placing implicit trust in the cloud provider's ability to patch the kernel, secure the container runtime, and prevent noisy (or malicious) neighbors from impacting your workload. This adds another layer of security risk that is often glossed over in the rush to adopt "cloud-native" solutions.

Containerization encouraged very bad patterns like microservices because containerization itself IS a bad pattern
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The microservices craze is inextricably linked to the rise of containerization. Containers made it *easy* to chop up applications into dozens of tiny, independently deployable pieces.

But "easy" does not mean "good."

This pattern traded the manageable, internal complexity of a monolith for the distributed systems nightmare of microservices. We now have to deal with service discovery, network latency, distributed tracing, and eventual consistency, all because containers made it convenient to run 100 processes instead of one.

Containerization didn't solve an architectural problem; it **enabled** an architectural problem by making a bad pattern deceptively simple to implement.

The docker flavour of containerization pretends to be a transient OS, while transient OSes don't exist
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Docker images act like mini-operating systems, bundling their own file systems and libraries. The community promotes the "cattle, not pets" mantra, suggesting these are transient, ephemeral entities.

But **production applications are not transient**. They have state. They generate logs. They need stable storage.

This "transient OS" concept is a lie. It creates a fundamental conflict: we have stateful applications running inside a system designed to be stateless, leading to complex "solutions" like persistent volume claims and external log shippers, which are just hacks to work around a flawed premise.

Docker's layered fs is an inefficient monstrosity of useless and unpredictable data that's going to fill up your storage with junk sooner or later
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Docker's layered file system (like OverlayFS) is a clever solution for development, saving disk space during the *build* phase.

In production, it's an inefficient monstrosity. Every write to a container triggers a **copy-on-write operation**, which adds I/O overhead. Worse, it's a storage management nightmare. Unused layers, dangling images, and build caches pile up, **filling your host's storage with gigabytes of unpredictable and useless junk**. Cleaning it up (`docker system prune`) becomes a mandatory, and often risky, maintenance chore.

One of the recent trends to address these kinds of issues is using minimal base images like Alpine and others. But hey, it does defeat the purpose of Docker, don't you think?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is the ultimate irony. The initial promise was "it works on my machine" because *all* dependencies were bundled in a fat, OS-like image (e.g., `ubuntu:latest`).

When we realized these images were bloated and insecure, the community pivoted to minimal base images like Alpine Linux. But this admission of a problem defeats the original purpose!

Now, we use Alpine, which uses `musl libc` instead of the standard `glibc`, introducing **subtle and maddening incompatibilities**. Worse, these minimal images strip out all the essential debugging tools (`bash`, `ps`, `curl`, `netstat`). When a problem occurs in production, you can't even `exec` into the container to find out why. You're left flying blind, all in the name of solving a problem (bloat) that containers themselves created.

Containers are not spawning a real init system but they are supposed to run just one process: isn't it too much for just running a process?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is the "One Process" fallacy.

A container doesn't boot a real init system; it just launches your application as PID 1 (which brings its own set of problems, like handling zombie processes).

Think about the overhead: an entire OS userspace, a virtual network stack, and a layered filesystem... all just to run a **single process**. It's an enormous amount of abstraction for a very simple goal. This design choice is also what makes debugging a nightmare.

Isn't it hard to debug? Why sidecar containers are even a reality?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Yes, it's incredibly hard to debug. And the "solution" is the **sidecar container**, which is the final proof of this model's failure.

Need to run a logging agent? A monitoring agent? A service mesh proxy? You can't run it in the *same* container (that would violate the "one process" dogma).

So, we have to launch and manage an **entirely separate container**—with its own filesystem and process space—just to run a helper process that, on a normal server, would have been a simple, lightweight agent. The very existence of the sidecar pattern is the ultimate admission that the container model is fundamentally broken for production workloads.

Where Containers still make sense in Prod
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Any use cases that are still uncovered by other isolation techniques (eg. GPU in AI workload): but it is still a matter of time, other isolation techniques will cover new use cases
2. When you can't just refactor a legacy app and you need to move it in a clustered/distributed system

In any other case I believe we are going to see apps moved away from containers.


It's Time to Rethink Production
-------------------------------

We've been sold a narrative that containerization is the only path to modern infrastructure. It's not.

It's a fantastic development tool that we've forced into a production role, and we are paying the price in complexity, insecurity, and inefficiency.