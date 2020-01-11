# Domain Driven Design (DDD)

[Domain-Driven Design (DDD)](https://en.wikipedia.org/wiki/Domain-driven_design) is an under utilized concept in software engineering. While generally useful, it's jet fuel in the engineering afterburner when used in conjunction with languages that feature immutability by default, strong type systems, and functional paradigms. That's somewhat ironic given that DDD came into existence specifically to address the complexity of developing Object Oriented software - not functional. As it turns out, DDD is exceptionally well suited for message oriented architectures (functional or otherwise) such as trading systems. What follows is a surface level introduction to DDD. See [Domain-Driven Design (DDD)](https://www.amazon.com/gp/product/0321125215?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0321125215) for a full treatment, and [Domain Modeling Made Functional](https://fsharpforfunandprofit.com/books/) for a great overview of mixing DDD with functional paradigms.

In what follows we define a software system as an embodiment of hardware and software built by an organization to fulfill some business purpose.

Domain Driven Design pays dividends in two principal areas:

1. The software development process itself. DDD can vastly increase engineering velocity, help orgs scale with fewer anti-network effects, empower individual contributors, and reduce variability in project time/difficulty estimation.
2. Software quality and system reliability. Modeling errors (under the broader than usual definition that we'll subsequently discuss) often account for the most business impactful system failures. They're also the hardest to prevent within the engineering org alone as modeling errors are the result of miscommunication between domain experts and engineers.
