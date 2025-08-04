.. title: The Hidden Costs of Static Linking and Containerization: A Critical Analysis
.. slug: hidden-costs-static-linking-containerization
.. date: 2025-02-02 07:00:00 UTC
.. tags: containers, static-linking, development, security, infrastructure
.. category: Development
.. description: An analysis of why static linking and containerization may be harmful for IT ecosystem
.. category: Development
.. author: Alessandro Bolletta


Statically-linked Programs are the evil
---------------------------------------

The trend toward static linking represents a fundamental regression in software engineering principles. By bundling every dependency directly into the executable, we're not just bloating our binaries - we're actively dismantling decades of progress in software modularization. Each statically linked program becomes an island, disconnected from the ecosystem of shared improvements and security updates.

Consider what happens when a critical vulnerability is discovered in a commonly used library. In a properly designed system using shared libraries, a single system update would protect all applications simultaneously. Instead, with static linking, we must embark on a complex and error-prone process of identifying every affected program, rebuilding each one individually, and ensuring they all get redeployed.

.. TEASER_END

Containerized Programs are the Evil (with an uppercased E at the beginning)
---------------------------------------------------------------------------

If static linking is problematic, containerization compounds these issues exponentially. Not only do we face the fundamental problems of static linking, but we now multiply them across isolated environments. Each container becomes its own separate universe with redundant copies of shared resources, creating a nightmare of inefficiency and complexity.

The storage implications alone are staggering. Where a traditional system might maintain one copy of a commonly used library, a containerized environment duplicates this library across potentially hundreds of containers. This multiplication effect extends to memory usage as well - shared libraries can no longer be shared across container boundaries, leading to massive memory bloat as each container loads its own copy of identical code.

Deployment times suffer significantly as entire container images must be transferred across networks. What should be a simple application update becomes an exercise in moving entire system images. The orchestration requirements grow exponentially, requiring complex management systems just to keep track of all these isolated environments.

Both produce unsafe applications and expose you at risks
--------------------------------------------------------

Static linking and containerization create dangerous security time bombs by embedding frozen versions of libraries into applications, making it impossible to quickly patch discovered vulnerabilities. When a critical security flaw is found in a shared library, systems using dynamic linking can deploy a single patch to protect all applications simultaneously, while statically linked applications require individual rebuilds and redeployments - a process so complex that vulnerable versions often remain in production indefinitely. Containerization worsens this problem by bundling entire operating system environments, expanding the vulnerability surface exponentially and making security audits nightmarishly complex. Each container potentially includes thousands of packages that need individual tracking and updating, creating an unmanageable maze of dependencies. The false sense of security provided by containerization leads to dangerous complacency, as developers mistakenly believe container isolation protects them from security vulnerabilities while sharing the same kernel. Sensitive information like API keys often gets baked into static binaries or container images, making secret rotation difficult and increasing exposure risks. The solution lies in transparent, centrally managed dependencies that enable rapid security responses, not in the isolation and rigidity that static linking and containerization impose.


Why We Do Love to Statically Link and Why There Are Hidden Costs Behind It
--------------------------------------------------------------------------

The seductive appeal of static linking lies in its promise of deployment simplicity and dependency predictability. Developers often embrace it because it seems to solve the "it works on my machine" problem - when everything is bundled together, environmental differences supposedly disappear. However, this apparent simplicity masks a deep complexity that manifests in numerous hidden costs.

The maintenance burden becomes particularly apparent over time. Security updates, which should be straightforward, turn into massive undertakings. Each statically linked program must be individually identified, rebuilt, and redeployed. Version tracking becomes exponentially more complex as each program effectively contains its own complete dependency tree. The testing burden multiplies because each static build represents a unique combination of library versions that must be verified independently.

Resource inefficiency manifests in multiple ways. Beyond the obvious waste of disk space from duplicate code, there's a significant memory overhead when multiple programs load their own copies of identical libraries. Build times increase dramatically as each program must compile all its dependencies from scratch, and deployment sizes balloon with redundant code.

Why the C Language is Really Good in Making Software Reusable
-------------------------------------------------------------

The C language's approach to software reuse through shared libraries represents one of its most sophisticated yet underappreciated features. At its core, C's ability to create stable Application Binary Interfaces (ABIs) enables a level of code reuse that modern approaches often fail to match. This isn't just about saving disk space - it's about creating a genuine ecosystem of reusable components.

The straightforward linking model in C allows programs to dynamically load code at runtime, enabling true resource sharing between applications. When multiple programs use the same library, they share the actual memory pages containing that code, significantly reducing the system's memory footprint. This sharing extends beyond mere storage efficiency - it enables system-wide optimizations that benefit all applications simultaneously.

Furthermore, C's model enables clear separation between applications and their dependencies while maintaining high performance. Platform-specific optimizations can be implemented at the library level, automatically benefiting all applications that use that library. This model has successfully powered Unix-like systems for decades, demonstrating its effectiveness in real-world deployments.

It's ridiculous to find how a language from 1970 offers better code reusability than the ones from 2020s. 55 years ago some wise people envisioned very smart concepts that are still foundational.

Why an Operating System That Doesn't Reuse Libraries is Not an OS by Definition
-------------------------------------------------------------------------------

The fundamental purpose of an operating system extends far beyond merely managing hardware resources - it serves as a platform for sharing and coordinating resources among applications. When we bypass this through static linking and containerization, we're essentially creating multiple parallel "operating systems" within one host, fundamentally undermining the very concept of what an operating system should be.

Consider the primary responsibilities of an operating system: it must efficiently allocate system resources, manage shared memory, and coordinate access to hardware. These functions aren't meant to be isolated - they're designed to create a cooperative environment where applications can share resources efficiently. When each application brings its own copy of system libraries, we're not just duplicating code - we're fragmenting the very foundation of system resource management.

The provision of shared services represents another crucial aspect of operating system functionality. By maintaining common libraries available to all applications, the operating system creates a standardized platform for development and execution. This standardization isn't just about convenience - it's about creating a reliable, secure, and efficient computing environment. When applications bypass these shared services through static linking, they're effectively declaring independence from the operating system's core functionality.

Using Shared Libraries Would Make Programs More Modern, Testable, Efficient
---------------------------------------------------------------------------

The adoption of shared libraries aligns perfectly with modern software development practices, offering advantages that go far beyond simple resource sharing. Consider the impact on modularity: when programs rely on shared libraries, they naturally develop cleaner interfaces and better separation of concerns. This modularity isn't just theoretical - it manifests in practical benefits throughout the development lifecycle.

Testing becomes substantially more manageable with shared libraries. Instead of testing entire static binaries as monolithic units, developers can focus on testing the interactions between their code and well-defined library interfaces. This approach not only reduces testing complexity but also helps identify issues more precisely. When a bug is found in a shared library, fixing it benefits all applications simultaneously.

The efficiency gains from shared libraries extend throughout the entire software lifecycle. Build times decrease significantly when programs don't need to compile their dependencies from scratch. Deployments become faster and more reliable when they can rely on existing system libraries. Memory usage improves as multiple programs share the same library code in memory, and cache utilization becomes more efficient when common code paths are shared across applications.

The Only Modern Revolution I See Coming is Stateless OSes
---------------------------------------------------------

The future of operating systems lies not in static linking or containerization, but in embracing a truly stateless architecture that revolutionizes how we think about system resources and application dependencies. This vision combines the best aspects of immutable infrastructure with the efficiency of shared resources, creating a new paradigm for system design.

At the core of this revolution is the concept of an immutable system layer containing verified, signed dependencies. These system libraries would be read-only and atomic in their updates, eliminating many traditional points of failure in system maintenance. However, unlike current trends toward static linking, these libraries would be shared efficiently across all applications.

The user space in such a system would be truly dynamic, with intelligent library loading and version management. Instead of each application maintaining its own copies of libraries, the system would efficiently manage shared resources, loading different versions only when absolutely necessary and optimizing memory usage across all applications. This approach would maintain the isolation benefits that developers seek from static linking while eliminating the resource waste and security risks.

Security in this new paradigm would be fundamentally different from current approaches. Instead of relying on the isolation of static linking or containers, security would be built on a foundation of verified execution paths and centralized vulnerability management. System-wide security patches could be applied instantly and atomically, eliminating the complex coordination currently required to update statically linked applications.

The revolution isn't about finding new ways to isolate our code - it's about creating intelligent systems that can dynamically manage dependencies while maintaining security and efficiency. By moving away from the current trends of static linking and containerization, we can build systems that are truly efficient, secure, and maintainable. The future lies not in isolation, but in intelligent sharing of resources and code.

Containerized or statically-linked applications should be embraced only for playground or very small deployments
----------------------------------------------------------------------------------------------------------------

While I find useful using containers or statically-linked applications to build toy projects, and I am going to use them for these purposes, they cannot certainly be a reference model for the future.

snap, flatpak, AppImage, Docker should not be taken so seriously as they pretend to be.