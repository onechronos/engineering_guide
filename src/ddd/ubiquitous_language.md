# The Ubiquitous Language

Core to DDD is the notion of a [Ubiquitous Language](https://martinfowler.com/bliki/UbiquitousLanguage.html) that's used when developers speak to domain experts (not developers) about the system and its underlying components. Such a language consists of "nouns" such as "Customer" and "verbs" such as "UpdateCustomer."  Although the phrase Ubiquitous Language found initial use in the context of developers talking to domain experts and other stakeholders, good DDD uses Ubiquitous Language as a conceptual foundation for the design of software itself. As such, establishing the Ubiquitous Language should be the first step in designing any business system (software or otherwise).

DDD relies on the Ubiquitous Language being a well defined, internally consistent, unified, and contradiction free model of the overarching system. For example, if your Ubiquitous Language has the notion of User, every stakeholder in the company should agree on the precise semantics of what a User is, and all software subsystems should have a conceptually identical notion of a User.

Unsurprisingly, that's not always practical or even possible for real-world systems. "User" might have a different meaning across business departments or software subsystems. For example, an engineer or business stakeholder might say "OneChronos operates a portal where users can change port settings, run "what-if" analysis on historical auction results, and submit computational bidders." That sentence contained five items that we must define within our Ubiquitous Language:

* Portal
* User
* What-If Analysis
* Historical Auction Result
* Computational Bidder

Each of those items brings with it ambiguity:

* Portal: the public website that anyone can access, the one that clients with logins can access, or the one that OneChronos employees use to watch trading systems?
* User: the people that we bill? The people logged into the portal (which portal?) Traders submitting orders on their own behalf? Brokers?
* What-If Analysis: the queries run by Users (oops, define User) interesting in seeing how alternative Computational Bids would have filled? OneChronos engineers testing alternative solvers?

... etc etc.

Two approaches to resolving this ambiguity exist:

* Use conjugation to make every word in the Ubiquitous Language unique and unambiguous. Instead of ```Portal```, say ```ClientPortal```. Instead of User, say ```NonBrokerClientPortalUser```.
* Establish [Bounded Contexts](https://martinfowler.com/bliki/BoundedContext.html) that break up the larger system into smaller ones, each with their own Ubiquitous Language. In this approach, a Client Portal Context might have a ```Subscriber``` and a ```User```. Their meaning is ambiguous when the context is unknown, but unambiguous within a context.
