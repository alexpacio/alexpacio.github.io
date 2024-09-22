.. title: Full-fledged API + e2e tests + benchmark + IaC + Helm charts + more as an (interesting) exercise!
.. slug: python-k8s-api
.. date: 2024-09-22 14:19:03 UTC+02:00
.. tags: k8s kubernetes fastapi pulumi k6 cloud-native
.. category: boilerplate
.. link: https://alexpacio.github.io/posts/python-k8s-api/
.. description: A cloud-native application packaged in an all-in-one Helm redistributable package. Boilerplating has never been so funny :)
.. type: text

Last week, I was contacted by a recruiter who asked me to create a project for a hiring challenge. The project seemed interesting, so I decided to take it on. At the very least, I would learn something new, which I was eager to explore: Pulumi, k6, FastAPI and some fancy modern things that make you a cool dev!

The project involved creating a simple REST API in Python, which needed to be packaged with Helm, ready for deployment in a Kubernetes (K8s) cluster, and including all the essential tools required.

**Requirements**

- A Python REST API backend program. The choice of framework was up to me
- Unit and e2e tests
- Swagger/OpenAPI documentation
- A CDK-like deployment script to automate the API’s dependencies: Terraform or Pulumi (for an AWS CDK-like experience). Since I was curious, I chose Pulumi
- Backend API should store its state via AWS PaaS services. Either a real AWS account or an automated way to use Localstack, a tool that simulates AWS’s APIs in your local environment, would have worked
- A Helm package to deploy the Kubernetes cluster
- Health checking mechanisms
- Scripts to tie everything together, making the cluster reproducible by following a series of steps outlined in a README file.
- Horizontal autoscaling
- Built-in application benchmarking

The ultimate goal is now clear: creating a self-contained package that could easily reproduce a complete, production-ready REST API backend in a cloud native and scalable setup.

**Result**

Here’s the result of my work: https://github.com/alexpacio/python-webapp-boilerplate

Although there are a few rough edges and minor aspects that I handled in a superficial way, this serves as an example of how a modern, asynchronous FastAPI backend can be delivered within a single Git repository. 
I believe it's a solid boilerplate to start with, and it allowed me to explore new tools like Pulumi, FastAPI, and other recent technologies.

**Thoughts**

Even though I feel like the entire AWS ecosystem is increasingly filling gaps (perhaps revenue gaps?) with bloat everywhere, I’ve been impressed with how concise, easy to use, develop, and reproduce a project like this can be. From this foundation, you can build a very sophisticated system while keeping everything in one place.
Additionally, you can test your AWS services locally and connect them later by simply using the Helm values file for the infrastructure or the .env file in the root folder for the application properties.

As mentioned, there’s definitely room for several small adjustments to make it work seamlessly, but I think this is a very solid and complete starting point! I may have missed something important, as I dedicated only a small portion of my time to this challenge.

Imagine having a temporary Kubernetes (K8s) namespace just to run your end-to-end (e2e) tests, reproducing the entire AWS PaaS stack and your application in a replicated and horizontally scalable way: you run the tests, collect the report, and then dispose of the environment.

The same approach can be used for your benchmarking needs.

With every change you make to your code, you can potentially test each step in your local cluster, which is automatically initialized in every aspect.

Just focus on your infrastructure and application code, set your environment variable files, and your portable cluster is up and running!

**Outcome**

I think I’ll continue to work on this. Cloud-native applications that avoid being locked into a specific cloud provider while still being distributed, scalable, and easy to use are possible.
I look forward to building an opinionated version of this soon. Stay tuned!

**Contribute!**

Feel free to use it as you wish, and let me know if you have any comments.

Cheers!