.. title: On the Phenomenology of Coding Agents
.. slug: on-the-phenomenology-of-coding-agents
.. date: 2025-08-24 10:00:00 UTC
.. tags: ai, coding, development, llm, programming
.. category: technology
.. description: An analysis of coding agents' capabilities and limitations in software development

Large Language Models have undoubtedly revolutionized how we interact with technology. They are, at their core, supercharged search engines with remarkable encoding and decoding capabilities. You speak to them in natural language, and they respond with perfect written text, flawless speech synthesis, or sophisticated image generation. Their utility is undeniable, yet we must acknowledge a fundamental truth: they cannot think in the human sense of the word.

.. TEASER_END

This limitation becomes particularly evident when examining their application in software development (web software development, precisely), where the divide between frontend and backend development reveals fascinating insights about the nature of LLM capabilities and constraints.

The Frontend Phenomenon
------------------------

Consider the case of frontend development, where LLMs like Claude Code demonstrate remarkable autonomy. I approach this from the perspective of someone who primarily works in backend development, with only rudimentary knowledge of HTML5, CSS, and JavaScript from my younger days, and virtually no experience with React. Yet through experimentation, I've discovered that this minimal foundation is sufficient to create reasonably complex frontend applications using modern frameworks, despite having no practical knowledge of React, shadcn, Tailwind, or Vite.

This phenomenon makes perfect sense when we consider the nature of frontend development. It is largely a discipline built upon reusable components, standardized message passing between clearly defined elements, and interfaces that must adhere to established conventions. The frontend ecosystem has evolved around clear patterns that existed long before the advent of LLMs. There are well-defined approaches to UI design, component architecture, state management, and routing that follow predictable structures.

The relative rigidity of these patterns actually works in favor of LLMs. When developing user interfaces, there is less room for creative deviation compared to backend systems. The business logic is typically abstracted away to the server side, leaving the frontend to focus on presentation, user interaction, and data flow according to established guidelines.

The Backend Complexity
-----------------------

Backend development presents a markedly different landscape. Here, the need for flexibility and custom solutions becomes paramount in ways that frontend development rarely demands. While you can certainly use services like Supabase or InstantDB for backend functionality, these solutions tie you to their specific Platform-as-a-Service offerings. In contrast, a React frontend generated with Claude Code can run entirely in your local environment without external dependencies.

Backend systems require an elasticity that frontend applications, in perhaps ninety-five percent of cases, simply do not need. The business logic, data processing, system integrations, and performance optimizations that characterize backend work often involve unique combinations of requirements that cannot be easily templated or standardized.

Of course, there are exceptions to this frontend simplicity. When you need to create sophisticated applications that perform real-time graphics rendering, decode video streams using WebCodec APIs, or implement complex WebRTC communications, the frontend becomes just as challenging as any backend system. In such cases, LLMs prove equally inadequate at providing meaningful assistance.

The Dataset Limitation
-----------------------

This brings us to a crucial understanding about how LLMs function in coding contexts. These models do not invent solutions; they propose approaches that already exist within their training data. The probability that a frontend problem has been encountered and documented before is significantly higher than the likelihood that a custom backend solution exists in the model's dataset.

The fundamental issue is not necessarily about innovation in the absolute sense, but rather about context and customization. When a project requires a creative process that must account for multiple contextual elements to achieve success, and particularly when the code needs significant customization, LLMs inevitably find themselves forced to attempt connecting pieces of information without truly understanding their relationships. They lack knowledge of the wirings, the proper sequencing of components, unless they have witnessed these exact combinations during their training phase.

This limitation becomes apparent when dealing with projects that demand careful consideration of numerous interconnected factors, custom business logic, or unique architectural constraints. The models excel when they can draw upon established patterns and well-documented solutions, but struggle when they must synthesize disparate elements into cohesive, context-aware implementations.

Beyond the Human Brain Metaphor
--------------------------------

We must awaken from the dream that equates LLMs with human cognitive abilities and instead deploy them in domains where they can work reliably and effectively. The technology excels in areas where pattern recognition and existing knowledge synthesis provide value, but struggles when genuine creativity or novel problem-solving is required.

LLMs demonstrate remarkable proficiency in linguistic translation tasks and bureaucratic work. I envision significant potential for these systems in government offices handling paperwork, assisting lawyers with document preparation, supporting judges in managing administrative documentation, or helping surveyors and architects with routine technical writing.

In software development, their utility remains constrained to specific scenarios where the problems are well-documented, the solutions follow established patterns, and the creative requirements are minimal. They serve as powerful tools for accelerating development when working within familiar territories and established workflows, but they cannot effectively handle situations that require understanding complex interdependencies or custom wiring of components that they have not previously encountered in their training data.

I have found them particularly useful for building frontend applications, scripting, and Infrastructure as Code (IaC) tasks such as Terraform and other DevOps tools. However, they still struggle when building something that lies outside their training dataset or when the wiring process needed to connect the pieces together is complex enough that they cannot keep up with the challenge.

The phenomenology of coding agents reveals a technology that is simultaneously impressive and limited, capable of remarkable feats within defined boundaries while remaining fundamentally constrained by the scope of human knowledge that has been codified and made accessible during their training. Understanding these boundaries is essential for leveraging their capabilities effectively while avoiding the disappointment that comes from expecting them to transcend their fundamental nature as sophisticated pattern matching systems.

Finally, it is obviously ridiculous to even think that LLMs will be one day able to produce correct system software, like kernel modules or fundamental user-space applications. There should be no discussion about that but still there is some noise out there from who doesn't know this :)