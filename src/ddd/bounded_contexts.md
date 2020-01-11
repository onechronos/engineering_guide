# Bounded Contexts

At first glance the choice between the two options might look like one of semantics. In the "worst case," Bounded Contexts result in the need for the same number of words as a single Ubiquitous Language. Given that, why should we use them? The benefits are two fold:

1. Bounded Contexts form natural course-grained system boundaries. Such boundaries are desirable when engineering teams need to function in isolation, and when the systems within a Bounded Context need to evolve independently.
2. The use of Bounded Contexts can reduce cognitive load. Within a Bounded Context stakeholders can use whatever language is the most natural without concern for how domain experts are using language elsewhere. Accounts Payable will have a different notion of a Customer than the Exchange Connectivity Team, and within a Bounded Context, that's fine.
