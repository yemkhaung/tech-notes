# Microservices Overview

### *Why do we need Microservices?*

In tranditional **monolitic** applications, it is more simple to architect and develop the application, but every component are tightly coupled in one big application. Every changes, failures in one component will affect entire application.

Thus, will have troubles, head-aches in cases,

1. **Not flexible** : Adoption of new performant, productive languages, frameworks is a lot of work since entire codebase and architecture needs to change.
2. **Slow Development** : developers need to understand large codebase for maintaining features
3. **Blocks continuous deployment** : one change in a component needs to redeploy entire application
4. **Unreliable** : peak traffic in one component or bugs in one component will affect entire systems performance, failure
5. **Unscalable** : Resource requirement in one component (like more computing power, more memory) will lead to scaling entire instance of application. Cannot make selective scaling to specific components thus wastes cost and resources.

**Microservices**, solves above problems.

### What are Microservices?

Microservices are **small**, **isolated** services that represent a portion of your *business domain*.

### Microservices Architecture

* User Interface
* API Gateway
* Services
* Management
* Service Discovery
* Message Queue

