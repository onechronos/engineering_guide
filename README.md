# Engineering at OneChronos

## What Makes Trading Venues Special

The software systems underlying trading venues (henceforth referred to informally as exchanges) differ from most application domains in a surprising way: their simplicity. Software of even moderate complexity often involves concurrency, side effects, external dependencies, compatibility requirements spanning two or more operating systems, and some form of a visual user interface. Core trading matching systems, by contrast are serial (within system boundaries), pure (when implemented as such), free of external blocking dependencies, server-side, bound to a single environment, and sans UI. They are message oriented systems that map a well defined set of inputs to outputs.

That's not to say that exchanges are *easier* to build than other software systems - they come with their own share of unique challenges. Notably, their economic significance qualifies them as a [safety-critical system](https://en.wikipedia.org/wiki/Safety-critical_system) and their demanding performance requirements puts them in the domain of [real-time computing](https://en.wikipedia.org/wiki/Real-time_computing). Both are extreme technical challenges.

That said, given the right legwork up front, exchange architectures are shockingly easy to build, maintain, and evolve. This document details the engineering practices of OneChronos, which are themselves designed to produce highly performant, highly correct software at a rapid pace. Our Systems Engineering guide details the architectural patterns and primitives upon which complex, business-centric applications can emerge and evolve. Taken in conjunction, they're a guide to building software at OneChronos.

The software that we produce today is for matching trades, but these patterns and practices are universal when it comes to authoring software for safety-critical event driven systems. All future ambitions involve software and hardware in this space, and our development process reflects that.

## Our Approach to Engineering

Before getting into the specifics of the software that we build, let's first discuss how we build it. Key tenants include:

* Rigorous Domain Driven Design
* Polyglot pragmatism
* Anti bikeshedding tools and standards
* An approach to testing that emphasizes formal methods, property testing, and end-to-end testing
* A rejection of methodological development practices, e.g. scrum

The above isn't a magic recipe for writing software in general. It's a recipe for writing the types of software that powers OneChronos. OneChronos views technology as a core differentiator. We operate within a highly competitive landscape but have two key advantages over incumbents:

* The ability to build systems blank slate, unencumbered by forty years of legacy
* An order matching paradigm that's highly amenable to architectures that enable superior latency characteristics, richer business features, and a lower TOC

These key advantages can help us outmaneuver and outcompete institutions far larger than ours.

### <a name="ddd"></a> DDD, The Ubiquitous Language, and Bounded Contexts

[Domain-Driven Design (DDD)](https://en.wikipedia.org/wiki/Domain-driven_design) is an under utilized concept in software engineering. While generally useful, it's jet fuel in the engineering afterburner when used in conjunction with languages that feature immutability by default, strong type systems, and functional paradigms. That's somewhat ironic given that DDD came into existence specifically to address the complexity of developing Object Oriented software - not functional. As it turns out, DDD is exceptionally well suited for message oriented architectures (functional or otherwise) such as trading systems. What follows is a surface level introduction to DDD. See [Domain-Driven Design (DDD)](https://www.amazon.com/gp/product/0321125215?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0321125215) for a full treatment, and [Domain Modeling Made Functional](https://fsharpforfunandprofit.com/books/) for a great overview of mixing DDD with functional paradigms.

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

To approaches to resolving this ambiguity exist:

* Use conjugation to make every word in the Ubiquitous Language unique and unambiguous. Instead of ```Portal```, say ```ClientPortal```. Instead of User, say ```NonBrokerClientPortalUser```.
* Establish [Bounded Contexts](https://martinfowler.com/bliki/BoundedContext.html) that break up the larger system into smaller ones, each with their own Ubiquitous Language. In this approach, a Client Portal Context might have a ```Subscriber``` and a ```User```. Their meaning is ambiguous when the context is unknown, but unambiguous within a context.

At first glance the choice between the two options might look like one of semantics. In the "worst case," Bounded Contexts result in the need for the same number of words as a single Ubiquitous Language. Given that, why should we use them? The benefits are two fold:

1. Bounded Contexts form natural course-grained system boundaries. Such boundaries are desirable when engineering teams need to function in isolation, and when the systems within a Bounded Context need to evolve independently.
2. The use of Bounded Contexts can reduce cognitive load. Within a Bounded Context stakeholders can use whatever language is the most natural without concern for how domain experts are using language elsewhere. Accounts Payable will have a different notion of a Customer than the Exchange Connectivity Team, and within a Bounded Context, that's fine.

#### Why This Matters

In what follows we define a software system as an embodiment of hardware and software built by an organization to fulfill some business purpose.

Domain Driven Design pays dividends in two principal areas:

1. The software development process itself. DDD can vastly increase engineering velocity, help orgs scale with fewer anti-network effects, empower individual contributors, and reduce variability in project time/difficulty estimation.
2. Software quality and system reliability. Modeling errors (under the broader than usual definition that we'll subsequently discuss) often account for the most business impactful system failures. They're also the hardest to prevent within the engineering org alone as modeling errors are the result of miscommunication between domain experts and engineers.

#### DDD and the Software Development Process

Everything hard about developing and evolving software from a pure engineering standpoint amounts to understanding the problem domain. Given a perfect model of said domain, everything else about developing software is easy. For one, estimating the time required to build a new feature in a world where models are fully specified is straightforward. New features have no blockers or requirements that make implementing them "hard;" in a fully specified domain, all possibilities live within the domain awaiting implementation. As such, the existing systems on which they're built accounted for their future existence. Any new system built specifically to support a new feature will also prove straightforward. Why? That system is also blocker-free, given a recursive application of the antecedent argument.

This panacea is far from a reality. The problem domain is almost never known in entirety, and systems inevitably evolve in ways that are unforeseeable. As such, implementing new features is typically an exercise in modifying existing systems to accommodate new ones. Dependencies are often uncovered sequentially; this leads to "project dependency hell" whereby simple feature request turn into massive engineering undertakings, the scale of which is hard to predict upfront.

The transitive dependences unearthed are "unknown unknowns," and more often than not they result from a mix of purely technical challenges and model misspecification. Technical dependencies are of the form "we need to change __X__ to accommodate __Y__, but doing that would break __Z__, so now we need to refactor __X__ and __Z__. Oh, if we change the ```fooBar()``` method of __Z__, __A__ breaks...which in turn breaks..." Heading into to the project, you *might* know about the dependency of __X__ on __Y__, but the deeper the dependency chain gets, the less foreseeable issues become. Model misspecifications result from misunderstandings - teams delivering functionality as specified, but the spec itself being incorrect or deficient. These conversations are typically of the form "Oh, yea, actually the report needs to include this column too" or "Oh, when I said that customers should be able to do __X__, I didn't mean brokers." This can result in new dependencies, and a vicious cycle. When stakeholders aren't totally aligned on requirements, the problem is acute.

The best weapons for fighting dependency hell are the use of a Ubiquitous Language and thoughtful domain modeling/isolation. Involving all stakeholders in the design process and using a Ubiquitous Language is the best way to ensure that everyone is in sync about what the requirements actually are. Domain modeling and Bounded Contexts let systems evolve independently.

#### System Correctness and Reliability

In a truly broad sense, everything that goes wrong with software systems in a business impactful way stems from one of three root causes that OneChronos refers to as Type I, II, and III faults. Systems can and should be able to handle faults without failing in a way that impacts the business. When failures happen, one or more faults (most commonly, a chain of them) are to blame. DDD is an effective tool for directly reducing the frequency of faults, but more importantly, the frequency of failures. Put another way, what the business cares about is $P(failure)$; DDD reduces both $P(fault)$ and $P(failure|fault)$, which can in turn greatly reduces $P(failure)$. We start with a discussion of fault types:

* Type I: faults that result from programming and operational errors of the type that are reasonably knowable and avoidable. Examples of faults in this category include:
  * Straight up programming bugs, e.g. de-referencing a null pointer, introducing data race conditions, invalid casts, overflow/underflow, etc
  * Deploying software incorrectly
  * Provisioning or configuring hardware or backbone infrastructure e.g. DNS servers incorrectly
* Type II: faults that result from failures in software/hardware made or operated by parties external to the organization. Examples include:
  * Crashing after calling into an API function exposed by a third party library
  * Data corruption resulting from buggy device drivers or hardware operating out of spec
* Type III: faults that result from a system functioning as-implemented, but the implementation itself failing to capture either the higher level intent of the business or the programmer. Examples of Type III faults include:
  * Implementing a mathematical algorithm in a way that's faithful to the specification of the algorithm, but the algorithm itself being wrong for the task at hand
  * Implementing business logic in a way that's faithful to a specification, but not in line with domain expert intent

We'll defer the discussion of Type I faults to a later section. DDD *is* an effective tool for dealing with them - but not the most effective one. Type I faults are faults introduced by humans making mistakes, so anything that makes humans less error prone reduces the probability of Type I faults. Reducing cognitive load is widely known to make humans less error prone, and DDD reduces cognitive load. That said, humans will always be more error prone than automated, deterministic processes. As such, the best tools for fighting Type I errors are the automated ones that deserve a full treatment independent of DDD.

The ability of DDD to prevent Type II faults isn't obvious, but as it turns out, DDD is one of the best tools available for avoiding them. As for why, DDD pushes organizations towards architectures with explicit failure modes, isolation, redundancy, and recovery capabilities. That's a good thing in general, but crucial to ensuring the reliability of systems with unknown unknowns; the failure modes of external dependencies are often unknown unknowns.

Type III faults are often the most challenging and business impactful. Type I and II faults have to do with failures of software. Type III faults result from failures of the software development process, and almost always result in business impactful failures.

One core tenant of DDD is the separation of business logic from "plumbing and boilerplate." Business logic is code of the form:

###### Short Validation V1

```python
def is_short_sell_eligible(market_context, order):
    if market_context[order.instrument_id].is_short_restricted:
        return False

    if order.side == OrderSide.SELL_SHORT:
        return True

    return False
```

Plumbing and boilerplate is code of the form:

###### Short Validation V1 Boilerplate

```python
    class MarketContext:
        def __getattr__(self, instrument_id):
            if not isinstance(instrument_id, InstrumentID):
                raise ValueError
            ...etc
```

Ideally, stakeholders should be able to see and read business logic code without being programmers themselves. In this specific example, no one except for the development team implementing it needs to care about how ```MarketContext``` works internally. Stakeholders on the business side can read the code for ```is_short_sell_eligible``` and understand it after receiving the explanation that

```python
if market_context[order.instrument_id].is_short_restricted
```

means: "look up the current (broader market) conditions for the symbol corresponding to ```order``` and return ```true``` if the security is short restricted."

If we had mixed business logic with implementation we might have ended up with something like:

###### Short Validation V2

```python
def is_short_sell_eligible(order):
    try:
        ord_restriction_status = conn_pool.take(). \
            execute("SELECT * FROM tbl_mkt_status WHERE " \
                    "inst_id == {}", order.instrument_id)
        if not ord_restriction_status == 0:
            return False
    except DBConnectionError as db_conn_error:
        raise Execption("...")
    except Exception:
        raise

    if order.side == 5:
        return True

    return False
```

Reading this and understanding the core logic independent of the plumbing is a high bar, even for developers. As it turns out, both the original version and the co-mingled version contain two logical errors. Neither checks that the order had a valid locate, and neither accounted for the fact that ```OrderSide.SHORT_EXEMPT = 6``` *could* result in an order that's short eligible.

No amount of testing or even the use of formal methods would catch this. The programmer's mental model of the algorithm ```is_short_sell_eligible``` was wrong, so any tests or proofs of correctness implemented to verify it would instead verify that the *wrong* model was *wrong in the right way.*

DDD is not the lone approach when it comes to dealing with this problem. Every software development methodology imposes its own views on how to gather requirements and action them. DDD is notable in its emphasis on modeling the problem domain in a way that for business-readable code as opposed to a business readable model that's subsequently translated to code. This, and the use of a Ubiquitous Language that's understood by technical and non-technical stakeholders reduces the probability of requirements getting lost in translation.

#### <a name="ddd-ml"></a> DDD + ML Languages for a Better SDP

One of the key advantages of functional, strongly typed languages such as ocaml (ReasonML) is clear separation of data from functionality and the expressiveness of types when it comes to modeling said data and dependencies. Let's revisit the previous example by writing out the types that implicitly took part in ```is_short_sell_eligible```:

```reason
type instrumentID =
  | InstrumentID(int);

type orderSide =
  | Buy
  | Sell
  | ShortSell;

type order = {
  instrumentID,
  orderSide,
};
```

To read the above, four rules for reading the code form of nouns in the Ubiquitous Language will suffice:

1. Types represent the shape of data - they're nouns
2. Types consist of other types
3. Types that have ```|``` in them mean "one of these types"
4. Types enclosed in ```{ ... }``` means "all of these types"

Given that, and concrete examples of how to read type signatures, e.g.:

* ```instrumentID``` takes one form - an ```InstrumentID``` that contains an integer
* ```orderSide``` has three forms: ```Buy```, ```Sell```, ```ShortSell```; none of those forms contains anything else
* ```order``` contains an ```orderId``` and an ```orderSide```

With minimal background on how to read the language the business side could look at this code point out that ```ShortSellExempt``` is missing as an option for ```orderSide```, and that ```order``` should contain a field representing a locate.

Actioning this, the developer would then write:

```reason
type orderSide =
  | Buy
  | Sell
  | ShortSell
  | ShortSellExempt;

type locateRequired =
  | Yes
  | No;

type order = {
  instrumentID,
  orderSide,
  locateRequired: option(locateRequired),
};
```

and explain that there's a fifth rule: ```option``` means that something might or might not be present. At this point, the developer still hasn't written any business logic. The business surfaced the issue with the domain model before time went towards writing "doomed to fail" code.

Now that the business side and the developer have agreed on the shape of the data itself, it's time to agree on the verbs. Before writing the actual logic, the developer could sketch out function definitions for feedback:

```reason
let isSaleProperlyFlagged: order => bool;
let isSecurityShortRestricted: order => bool;
let isShortSellEligible: order => bool;

let isShortSellEligible = order =>
  isSaleProperlyFlagged(order) &&
  isSecurityShortRestricted(order);
```

A quick lesson on reading these type signatures would make it clear what's happening, e.g. ```isSaleProperlyFlagged``` is a function that takes an ```order``` as an input and returns ```true/false```. At this point, the business should question the ambiguity of what happens when ```isShortSellEligible``` receives a buy order. That would lead to the correction:

```reason
let isSell: order => bool;
let isShortSellEligible = order =>
  isSell(order) &&
  isSaleProperlyFlagged(order) &&
  isSecurityShortRestricted(order);
```

With the shape of the validation nailed down the developer can write:

```reason
let isSell = order => order.orderSide != Buy;

let isLocateGood = order =>
  switch (order.orderSide, order.locateRequired) {
  | (ShortSell, Some(Yes)) => true
  | (ShortSell, _) => false
  | (Buy | Sell | ShortSellExempt, _) => true
  };

let isSaleProperlyFlagged = order =>
  switch (order.orderSide, order.locateRequired) {
  | (Buy | Sell, _) => false
  | (ShortSell, _) => isLocateGood(order)
  | (ShortSellExempt, _) => true
  };

let isSecurityShortRestricted = order =>
  switch (order.orderId) {
  | OrderID(1234) => true
  | _ => false
  };
```

A quick explanation of pattern matching semantics can make ```switch``` statements almost as readable as types. Short of that the developer could walk a stakeholder through every match arm, explain the conditions that lead to it, and verify and ask "is the function returning the right value in this case?"

Both the process and result are superior to V1 of ```is_short_sell_eligible```. In contrast to the process that led to the business discovering issues with the validator after the developer spent time writing it, our functional/ML DDD process surfaced it *before*. That was immaterial in this trivial example, but early detection of modeling errors can often save weeks or months. Furthermore, every noun and verb came straight from the Ubiquitous Language - there's nothing that would appear foreign to either a technical or non-technical stakeholder.

The question becomes: how well does this approach scale to larger projects and non-trivial business logic? In short, exceptionally well. Real-world systems contain enormous complexity in the portions of the codebase that developers deal with, but human produced business logic typically looks like a slightly expanded version of ```isShortSellEligible```. Business logic can itself be large and complex, but it's typically decomposable using the same incremental approach that we used to arrive at the final version of ```isShortSellEligible```.

Decomposition and method extraction often results in code that cleanly separates the implementation details from the pure functional domain that makes up actual business logic. The module systems typical of ML languages allow for a clean separation between types, method signatures (verbs), and the concrete implementations of both. As such, it's possible to create a separate "repository" of pure business logic in which all stakeholders collaborate with high fidelity and minimal overhead.

We won't go into the other technical merits of the DDD functional approach here, but there are plenty.

### Engineering for Reliability

Much of what's written on software engineering focuses on Type I faults, and for good reason. Type I faults are the easiest to attack head on with the use of computer science itself. Type safe and memory safe languages prevent a large class of bugs, as does the use of advanced testing techniques such as symbolic execution and formal methods. Furthermore, the rigor and sophistication of DevOps increased massively in the past decade. Automated provisioning, containerization, and the notion of declarative "infrastructure-as-code" have made it far easier to avoid Type I faults in production.

Of note, the best tools for accomplishing the above are predominately available as FOSS. Historically, large companies devoted significant engineering resources to building such systems in-house, and the expertise/effort required to do so put them out of reach for most. Today's startups are well positioned to adopt such tools out of the gates, and with great effect.

The languages that we use for mission critical systems due to their strong emphasis on correctness, memory safety, immutability, and type safety (Elixir aside) are:

* [OCaml](https://ocaml.org/) / [ReasonML](https://reasonml.github.io/)
* [Rust](https://www.rust-lang.org/)
* [Elixir](https://elixir-lang.org/)

Here's a (non-exhaustive) list of the tools that we believe in for software analysis and verification:

* [z3](https://github.com/Z3Prover/z3): Low level theorem proving / SMT solving
* [Imandra](https://www.imandra.ai/): High level theorem proving / formal verification
* [TLA+](https://lamport.azurewebsites.net/tla/tla.html): Theorem proving / model checking geared towards state machines and concurrent systems
* [KLEE](http://klee.github.io/): LLVM symbolic execution
* [PyExZ3](https://github.com/thomasjball/PyExZ3): Python symbolic execution
* [American Fuzzy Lop](http://lcamtuf.coredump.cx/afl/): Instrumentation guided fuzzing
* [Valgrind](https://valgrind.org/) and [Helgrind](https://valgrind.org/docs/manual/hg-manual.html): Memory and thread error detectors
* [Syzkaller](https://github.com/google/syzkaller/blob/master/docs/syzbot.md): Kernel fuzzing
* [Clang-Tidy](https://clang.llvm.org/extra/clang-tidy/): C/C++ static analysis
* [Clang AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html): Safer C/CPP
* [JetBrains IDEA Static Analysis](https://www.jetbrains.com/help/idea/command-line-code-inspector.html): Static analysis for JVM languages
* [Typespecs](https://github.com/elixir-lang/elixir/blob/master/lib/elixir/pages/Typespecs.md): Type annotations for Elixir
* [mypy](http://mypy-lang.org/): Static type checking for Python
* [Hypothesis](https://hypothesis.readthedocs.io/en/latest/): Python property testing
* [qcheck](https://github.com/c-cube/qcheck): OCaml/ReasonML property testing
* [excheck](https://github.com/parroty/excheck): Elixir property testing
* [BurntSushi/quickcheck](https://github.com/BurntSushi/quickcheck): Rust property testing
* [junit-quickcheck](https://github.com/pholser/junit-quickcheck): JVM property testing

Tools that help automate infrastructure and reduce the probability of Type I failures in production include:

* [Terraform](https://www.terraform.io/): Infrastructure-as-Code
* [Ansible](https://www.ansible.com/) - Software and hardware provisioning and configuration
* [Vagrant](https://www.vagrantup.com/): Reproducible local development environments
* [Packer](https://www.packer.io/): Machine image builder
* [Docker](https://www.docker.com/): Containerization framework
* [Kubernetes](https://kubernetes.io/): Container orchestration and cluster management

### <a name="testing"></a>Testing

Testing is crucial to any software development process and key feature of any system engineered for reliability. Given the development process that OneChronos uses and the types of software that OneChronos writes, out testing process is somewhat unique.

#### Unit Testing

Extensive use of unit tests and Test Driven Development is "typical." We tend to deemphasize unit tests (with the caveats that will follow) and emphasize the use of languages, tooling, and development practices that prevent and catch the same types of bugs that unit tests would.

We arrived at this practice having noted the following. Unit tests are a great way for developers to check their assumptions about how the code that they're working on will perform *during* the development process. Used in this capacity, they're effective, and a productivity booster in any language. Their usefulness outside of authoring code is more nuanced. For dynamic languages, refactoring without close to 100% test coverage is a recipe for disaster. But for languages with a strong story around type safety, isolation, and immutability, their usefulness is dubious. Unit tests tend to catch the same shallow bugs e.g. type incompatibility and the use of undefined variables that these languages prevent. Given that every line of code written represents both time spent now and technical debt later, minimizing the use of unit tests makes sense.

Catching deeper, more structural bugs requires a different approach. Property testing and formal methods are a better tool for uncovering issues with algorithms and data structures. A developer writing a sort routine could write unit tests that check made-up scenarios and the corner cases that they think of. Doing so is often circular. The corner-cases that they think of are the ones that they would have accounted for in their sort implementation, so writing unit tests typically won't lead to new chains of thought. As such, unit tests are unlikely to uncover the corner-cases that weren't accounted for, aka bugs.

Property testing and formal methods are more likely to uncover bugs given that they elicit a different thought process. Imagine that you as a developer are writing a routine that sorts an input list. You want to make sure that the routine always produces an output list with the same length as the input. The unit testing approach might result in you writing:

```python
def test_empty_list_stays_empty(self):
    self.assertEqual(0, len(my_sort([])))

def test_list_with_elements_unchanged_in_length(self):
    test_list = [x for x in range(5)]
    self.assertEqual(len(test_list), len(my_sort(test_list)))
```

If the sort algorithm had a subtle bug that applied to lists of length one or two, your unit tests would miss that. Worse, they'd bring false confidence that the code was correct. Future developers would see that tests were in place and assume that a passing test indicated that their changes were good to go. This is a problem even if your initial code was bug-free!

A property test of the form:

```python
@given(s.lists(ListStrategy))
def test_list_length_unchanged(test_list):
    assert len(test_list) == len(my_sort(test_list)))
```

stands a much better chance of catching said bugs on day one, and would prove less fragile in the face of future changes. NB: property testing and fuzzing $\neq$ formal methods. They're random test case generation with (best case) some light static analysis to exercise interesting code paths. The effectiveness of property tests hinges on writing good "strategies" for generating test cases. Good property testing frameworks make this as easy as possible and far more declarative than cranking out unit tests, but they still require care to exercise corner cases.

As an experienced developer it's easy to think that you'd never miss trivial corner cases or make silly mistakes. The reality is, software is subtle and hard, and everyone misses corner cases. Fun fact: a bug in Java's implementation of binary search (written by the veritable Josh Bloch himself) went undetected for [over twenty years](https://en.wikipedia.org/wiki/Binary_search_algorithm#Implementation_issues).

Automated theorem provers and property testing frameworks are better at hitting corner cases than humans, but not because they're smarter. To the contrary, they're effective tools for finding bugs *because* they're dumb and pathological. Humans are smart enough to understand high level intent, but computers aren't. Bugs arise when humans make implicit assumptions and don't include those assumptions when writing down dumb, low level logic.

##### Closing Notes on Unit Tests

When to unit test:

* Whenever you're working with a dynamic language
* When doing so helps your development process
* When property testing or formal methods are prohibitively difficult to apply to the problem at hand

When to avoid unit tests:

* The code in question is business logic - we'll get to testing business logic later
* Whenever what you're testing for is implicitly or explicitly checked by linting or compilation
* Whenever property testing could hit the corner cases that you're aware of, and then some

When unit tests find use, make sure to use them in conjunction with code coverage tools. The following is a list of the preferred unit testing and code coverage frameworks:

###### Rust

* __Unit testing framework__: built-in
* __Code coverage:__ [Tarpaulin](https://github.com/xd009642/tarpaulin): Rust code coverage

###### OCaml

* __Unit testing framework__: [alcotest](https://github.com/mirage/alcotest)
* __Code coverage__: [bisect_ppx](https://github.com/aantron/bisect_ppx)

###### Elixir

* __Unit testing framework__: built-in
* __Code coverage__: [excoveralls](https://github.com/parroty/excoveralls)

###### Python

* __Unit testing framework__: built-in
* __Code coverage__: [coverage.py](https://coverage.readthedocs.io/en/coverage-5.0.2/)

###### Java

* __Unit testing framework__: [JUnit](https://junit.org/junit5/)
  * __Mocking__: [Mockito](https://site.mockito.org/)
* __Code coverage__: [Clover](http://openclover.org/): Java code coverage

#### Testing Business Logic

True unit tests are not the right approach to testing business logic. The reason why is simple: *real* unit tests map one-for-one to specific functions/classes/modules in software. As we saw in our [section on DDD + ML languages](#ddd-ml) sometimes (and ideally) data structures and functions map one-to-one to business logic, but this is not always the case.

Software projects often use unit tests to exercise business logic that involves more than one software "unit." This is an anti-pattern that's often indicative of broader code quality issues. On the pure software side, unit tests that exercise aggregates (in the DDD meaning of that word) instead of individual units often hint at the existence of something simpler and more modular.

On the business side, business logic should (ideally) be a pure data driven mapping of inputs to outputs. It's formalized in code, but exists independent of any particular programming language or construct. Notably, business logic exists independent of software constructs like argument construction and function calls. Unit tests fail to appreciate this; they mix programming language specific nuance with the business logic itself. That makes it difficult to collaborate on test cases directly with the business side and couples test cases to specific language/framework/implementation details.

As such, OneChronos favors specific flavors of [black-box testing](https://en.wikipedia.org/wiki/Black-box_testing) for exercising business logic. Agile refers to some of what we do as [behavior-driven development (BDD)](https://en.wikipedia.org/wiki/Behavior-driven_development). While we don't believe in dogmatic [test-driven development](https://en.wikipedia.org/wiki/Test-driven_development), we do believe in and use frameworks and paradigms that look like stories/spec tests.

The best means for effective black-box testing are highly situation dependent. FIX gateways, some APIs, and portions of the match logic lend themselves well to testing via [golden files](https://medium.com/@jarifibrahim/golden-files-why-you-should-use-them-47087ec994bf). In other situations, spec tests of the [rspec](https://rspec.info/) and [cucumber](https://cucumber.io/tools/cucumber-open/) variety are the way to go.

Regardless of the flavor of black-box testing used, tests should use language agnostic frameworks that leverage universal input/output formats over universal communication protocols e.g. json passed via stdin/stdout to invoke business logic and verify results. When done so diligently, systems are free to evolve with higher velocity and reduced risk.

#### End-to-End / Acceptance Testing

An ancillary benefit of the black-box approach is test composability. For example, a test that exists to verify the correct behavior of a FIX engine can often find use as an end-to-end test when composed with tests that verify the behavior of the market data facility, matching engine, and trade reporting systems. Doing so results in a higher velocity development process and less fragile test code.

As such, there's not a strong distinction between functional, integration, system, and acceptance testing at OneChronos. Tests tend to follow a uniform style and receive a *fuzzy, temporal* classification based on how and when they're invoked. For example, a test might run on a small component in isolation and be a functional test. If the same test found use as a stage in an end-to-end testing pipeline running in production, we'd refer to it as an acceptance test. That's not to say that some tests aren't purely functional or end-to-end.

Tools and paradigms like cheap cloud compute, Docker, Terraform, and Ansible, and the [Robot Framework](https://robotframework.org/) make these testing modalities possible. Historically the extreme difficulty of creating realistic replicas of a company's production software environment and the cost of manual testing made the use of end-to-end testing impractical for most, and something that (true to the name) occurred at the end of the SDP. That's no longer the case.

[Robotic Process Automation](https://en.wikipedia.org/wiki/Robotic_process_automation) and the notion of infrastructure and system configuration as code now makes the use of end-to-end practical throughout the development lifecycle. Doing so catches problems early and empowers developers to iterate aggressively. That helps ensure that systems at OneChronos never become "legacy."

#### Verification and Formal Analysis

In a perfect world, all software at OneChronos would be [verified](https://en.wikipedia.org/wiki/Formal_verification). That's not the reality; formal verification is often prohibitively difficult and expensive, and [not always possible](https://en.wikipedia.org/wiki/G%C3%B6del%27s_incompleteness_theorems). That said, formal methods are actually a productivity booster in certain cases. When the system under test is readily modeled as a [finite state machine (FSM)](https://en.wikipedia.org/wiki/Finite-state_machine) it's often easier to write out invariants than think though test cases. The same is often true for low-level proofs of equivalence, e.g. proving a compiler optimization correct. The proofs that power formal methods offer the highest guarantees of correctness possible, and as an added bonus, they're typically more compact than an exhaustive test suite. That said, they can be much harder to write.

![Hierarchy of verification effort](img/verification_techniques.png)

NB: this graph [^validation_graph] should include a dot below ad-hoc testing for "PL static analysis," e.g. linting and the use of "pedantic" compiler warnings

Our rules and guidelines from program validation are as follows:

* Shallow static analysis:
  * __Example tools__: clang-tidy, pylint, compilation with whatever the language specific ```-Wall``` is
  * __Example targets__: all software written at OneChronos
  * __Use__: always
  * __Good at__: catching typos and PL use bugs
  * __Bad at__: catching logic bugs, or anything more structural
* (Less) shallow static analysis:
  * __Example tools__: JetBrains IDEA, Clang Static Analyzer, FB Infer
  * __Example targets__: all software written at OneChronos
  * __Use__: always
  * __Good at__: catching slightly more involved bugs, e.g. NPEs, some instances of use-after-free, dead code
  * __Bad at__: catching logic bugs
* Ad-hoc testing
  * __Example tools__: unit and behavioral tests
  * __Example targets__: all software written at OneChronos
  * __Use__: always, subject to the guidelines above
  * __Good at__: documenting and verifying that a known set of assumptions hold
  * __Bad at__: verifying that assumptions hold in the general case
* Fuzzing:
  * __Example tools__: property testing
  * __Example targets__: all software written at OneChronos
  * __Use__: almost always, subject to the guidelines above
  * __Good at__: expanding coverage beyond ad-hoc testing
  * __Bad at__: verifying critical systems, unless supplemented with other, more powerful methods
* Concolic execution & white-box fuzzing
  * __Example tools__: instrumentation guided fuzzers such as AFL
  * __Example targets__: XDP code for network timestamp manipulation
  * __Use__: when practical
  * __Good at__: exercising code paths - something that's crucial for testing languages without strong memory and type safety; taking a starting set of behavioral test cases with invariants and verifying that the invariants hold for similar test cases
  * __Bad at__: scaling to large systems or demonstrating that invariants hold in general
* Symbolic execution: when strong guarantees of correctness are important but not required
  * __Example tools__: using z3 to trace program execution and exhaustively explore all code paths
  * __Example targets__: proxy bidder analysis
  * __Use__: when practical
  * __Good at__: exhaustively exercising code paths; symbolic execution is the deterministic but far less scalable cousin of concolic execution
  * __Bad at__: scaling
* Deep static analysis:
  * __Example tools__: walking a program's AST/CFG and using z3 to prove invariants along the way
  * __Example targets__: combinatorial bid analysis
  * __Use__: when strong guarantees of program correctness are necessary, but formal verification is out of reach
  * __Good at__: verifying that a computer program is crash-free/deadlock-free
  * __Bad at__: verifying that a computer program produces the intended result for all possible inputs
* Formal verification:
  * __Example tools__: theorem proving with Imandra/z3/TLA+
  * __Example targets__: verifying certain properties of proxy bidders and combinatorial bids; verifying the distributed consensus protocol used for running auctions
  * __Use__: when the situation at hand calls for the strongest guarantees of correctness possible
  * __Good at__: proving that the expected invariants hold for all possible program inputs
  * __Bad at__: scaling, or being easy to do

### At OneChronos, Software Development = Software Engineering

Writing code might or might not be *engineering.* Programming is and often should be an exercise in pragmatism - getting the task at hand done. That's fine for environments where the value of feature velocity outweighs the value of correctness - something that's true for most companies. That's not true for settings like OneChronos where mistakes are potentially catastrophic. [What happened at Knight](https://www.henricodolfing.com/2019/06/project-failure-case-study-knight-capital.html) is a cautionary tale. One should never *write code* in such environments; teams should work together to *engineer systems.*

The core differentiators between coding and engineering software are *planning*, *process* and *repeatability.* Planning is about figuring out what you want a system to do before you start implementing it. *Process* is all about making sure that all software gets built and deployed in a consistent fashion. Planning + process = repeatability.

Notably absent in this description of engineering is any prescribed formula for planning, or any fixed process. There's a reason for that. No development methodology is "one size fits all," even within an organization. In fact, given our strong emphasis on individual contributors the team is often one person working on a specific system in isolation. It doesn't matter how that person chooses to run their own development process, as long as they treat it like one and hit quality standards. When individuals team up for a project, they're free to collaborate as they see fit.

Given that teams are free to do as they please OneChronos needs a "meta process" to ensure uniform quality, interoperability, and readability. Keeping in line with our approach to everything else, the process is/will be mostly automated, and puts great trust in individual contributors.

The following are the hard and fast firm-wide rules for software engineering at OneChronos:

* All code must be tested and verified to the standards outlined in this guide's [section on testing](#testing).
* Take a DDD approach to designing and building systems.
* Take a polyglot approach to development and build systems in whatever language is the best fit. You can have a favorite language, but that should never be a factor in this decision.
* Favor safer and more performant languages whenever possible, but use the best tool for the job. For example, python is a slow and "dangerous" language, but way better suited for research tasks than OCaml or rust.
* All compiled code must compile warning-free with all warnings enabled.
* Use "low cost" static analysis tools when available.
* Automate everything involving build / test / deployment process.
* Don't bikeshed. That applies 10x when it comes to syntax and matters of personal taste. The OneChronos style guides might not make sense in every situation but the benefits of uniform use are twofold:
  * Code that appears uniform company wide is easier to read $\Rightarrow$ reduced cognitive load $\Rightarrow P(\text{mistake})$ 
  * Bikeshedding over style is a huge time waster
* Code must be in strict accordance with the following language specific style guides and tools:
  * OCaml
    * Style guide: [OCaml community standards](https://ocaml.org/learn/tutorials/guidelines.html)
    * Linter: refmt
  * ReasonML
    * Style guide: ```refmt```
    * Linter: ```refmt```
  * Elixir
    * Style guide: [elixir_style_guide](https://github.com/christopheradams/elixir_style_guide)
    * Linter: [credo](https://github.com/rrrene/credo) and [dialyxir](https://github.com/jeremyjh/dialyxir)
  * Rust
    * Style guide: [rust community style guidelines](https://doc.rust-lang.org/1.0.0/style/) and [rustfmt](https://github.com/rust-lang/rustfmt)
    * Linter: [rust-clippy](https://github.com/rust-lang/rust-clippy)
  * Python
    * Style guide: [PEP 8](https://www.python.org/dev/peps/pep-0008/) and [Google](http://google.github.io/styleguide/pyguide.html). When there are contradictions, favor PEP 8
    * Linter: [pylint](https://www.pylint.org/) and [mypy](http://mypy-lang.org/)
  * Java
    * Style guide: [Google](https://google.github.io/styleguide/javaguide.html)
    * Linter: [Jetbrains IntelliJ IDEA](https://www.jetbrains.com/idea/)
  * C/C++
    * Style guide: [Google](https://google.github.io/styleguide/cppguide.html)
    * Linter: [Clang-Tidy](https://clang.llvm.org/extra/clang-tidy/)
  * Javascript
    * Style guide: [AirBNB](https://github.com/airbnb/javascript)
    * Linter: [ESLint](https://eslint.org/)
  * CSS
    * Style guide: [AirBNB](https://github.com/airbnb/css)
    * Linter: [stylelint](https://github.com/stylelint/stylelint)
* You as a developer are free to disable linting and compilation warnings as needed, but note why you did so in a commit. Be as targeted as possible when disabling a warning, e.g. do so at the line level instead of the function level. Know that because everything builds and deploys programmatically, you're explicitly signing off on it going into production.

### At OneChronos, Software Engineering $\neq$ Research

Much of what we do at OneChronos is scientific research - not engineering. The distinction is that engineering is the practice of turning specifications into product under a well defined process. Research is an exercise in turning ideas that might or might be possible into engineering tools. Much of what's written on the software development process - planning and estimation in particular - does not apply to research.

Because research inherently focuses on the unknown, estimating the absolute time/cost difficulty of getting to a result doesn't make sense. What does make sense is a relative statement of opportunity cost. By researching __A__, you will not have the time or money to research __B__. A commitment to __C__ for two months will delay an engineering project __D__ by two months; given that the expected value of __D__ is __X__, is __C__ worth it?

When embarking on a new project it's crucial that all stakeholders understand when something is research as opposed to engineering. Engineering teams might field requests for features that aren't doable with the existing tools. Other requests might be best accomplished with research, but achievable with the existing options. Some requests will lead to straightforward exercises in implementation. Communicating what category the request falls into helps with resource planning and avoids scope creep. The worst case scenario is one in which a business stakeholder requests research (without knowing it) for something that's of marginal value.

Building a deep tech company is an exercise in doing enough engineering to keep the company moving in the right direction *now* and enough research to keep growth strong and competitors at bay one, two, five, and ten years from now. OneChronos has a culture of long term thinking, and investing in research is a big part of that. Communication and planning is key to hitting short term and long term goals in a way that maximizes terminal value.

### Effective Collaboration Between Engineering and Non-Engineering Stakeholders

Before discussing techniques for effective collaboration it's worth unpacking what we mean by stakeholder. Every company has internal and external stakeholders. Internal stakeholders include engineers, business development professionals, product owners, and the company itself. External stakeholders include customers, regulators, and vendors. Investors/non-employee shareholders lay somewhere in the middle.

Almost every initiative involves two or more stakeholders. Whenever that's the case, effective collaboration and a shared interest in the outcome is key to success. At times that collaboration is entirely commercial. Sometimes it's entirely technical. [DDD](#ddd) applies to both situations and everything in-between. Unlike most of the SDP religions, DDD is à la carte and has no methodological or technological underpinnings. Its core philosophy is simple:

* Effective communication requires that stakeholders establish a Ubiquitous Language to describe the problem domain
* Formal descriptions of process models should underpin design decisions
* Stakeholders should work together to seek the smallest, most conceptually accurate process model possible and put that model at the heart of system design

If this sounds like a concept that's more general than software, that's because it is. Although it wasn't conceived or marketed as such, we'd argue that DDD can apply to everything from corporate structuring to collaborating with customers on what product to build. We take this view when discussing the use of DDD at OneChronos.

Much of what's written on the SDP and DDD focuses on "technicals" collaborating with "non-technicals." There's often an implicit or explicit assumption that the "technicals" are engineers divorced from the nuances of product and the "non-technicals" have little to no idea how the product works on a technical level. At OneChronos, that's a bad mental model; the distinction between "technical" and "non-technical" is (by design) thin. Keeping that in mind, here's a high level workflow for effective collaboration; it doesn't shy away from having engineers learn more about the business than what's typical, or having domain experts read code:

1. Discuss the project at a high level: "we need a system to make sure that auctions always clear within the NBBO."
2. Establish and use a Ubiquitous Language. "What does clearing mean? What's the NBBO?"
3. Iterating on a process model. Make sure that engineers understand what the process is doing and domain experts understand how that will translate to a system specification. Engineers should look for ways to conceptually simplify the business process: "Do we actually need to stream the NBBO, or is a snapshot of it at the time of the auction good enough?" Domain experts should look for ways to simplify the system models put forth by engineers: "Should the matching engines be doing this calculation, or does it make more sense to have the market data gateways do it?"
4. Formalize business logic at the level of scenarios and input/result mappings. Do this in a format that's both human and machine readable.
5. Code the requisite data structures and function type signatures. Mock up a complete end-to-end workflow whenever practicable. Don't fill in business logic or worry about implementation details; instead, focus on data types and the transformations between them. For example, if the real system would call an external services or write to a file, simulate those effects with pure functions that return hard-coded data.
6. Show (in code form) domain experts the workflow. Walk through examples of inputs mapping to outputs. For a GUI, that might mean showing how toggling form components leads to different backend APIs being calls. For an order entry gateway, that might mean showing how messages flow in and out. Note that at this stage, the code shouldn't be doing anything real. Implementing the real functionality will take far more time and will be for naught if the underlying assumptions about how data flows are wrong.
7. GOTO 3) until everyone is in agreement that the domain model arrived at is a good one.
8. Fill in the business logic necessary to make the system work according to spec. To the extent possible, don't write any impure code that's not business logic. Make sure that the scenarios established in 4) are working.
9. Demo the business logic in the same fashion that you demoed the workflow. Iterate until stakeholders agree that all scenarios are properly handled. Add to Step 4) scenario bank as needed.
10. Ask all stakeholders the question: "Is this done?"

### Document Algorithms, Systems and Design Decisions - Not Code

Line level code comments for the sake of code comments aren't valuable. More often than not well named variables and type signatures are enough. When they're not, that's often cause for concern; rote code should be readable without commentary.

In general, comment code if one of the following is true:

* You're documenting an algorithm or something that's unreadable for a good reason, e.g. "The following is a trick for computing $x \text{ mod } n$ without division. It works because $n = 2^k, k \in \mathbb{Z}$. See [Compute modulus division by 1 << s without a division operator](https://graphics.stanford.edu/~seander/bithacks.html#ModulusDivisionEasy)."
* You're documenting a __stable__ API that will __definitely__ find use outside of the project that it lives in. Note the emphasis on "stable" and "definitely."
* You're documenting design decisions. These notes are often invaluable for understanding code and preventing future teams (or yourself) from wasting time on pursuing dead ends when revisiting code down the road.

Code comments aside, systems should come with extensive and actively maintained documentation. Every code project should contain a README explaining (at minimum):

* What the project does
* How to build, run, and deploy the project
* What the project's system level dependencies are

#### Process Automation is the Best Form of Documentation

High level instructions for building, running, and deploying a project are great. What's not great is an elaborate formula for doing so, presented in text form. Instead of writing out instructions, script a process. This could involve writing a Makefile, an Ansible playbook, or a Terraform module. When done properly, build/install/deploy scripts serve a dual purpose: documenting in a structured way while simultaneously automating laborious and error prone processes. 

### Avoiding Heavyweight Processes and the Agile Industrial Complex

The core tenants of agile are good ones. That said, the agile practices that most stakeholders encounter in the wild are what Martin Fowler refers to as faux agile, the use of which drives the [Agile Industrial Complex](https://martinfowler.com/articles/agile-aus-2018.html). Frustrations with heavyweight development processes gave birth to agile. Ironically, most companies that practice agile are in reality practicing something heavyweight, and far from it.

As we've already discussed how collaboration, communication, and process is key to building systems. What we don't like about faux agile is two fold:

1. Its heavy emphasis on fine grained resource planning and structure around meetings
2. The intermediation of engineers and stakeholders via project managers

Caveat emptor: our arguments for why both of these are bad is highly situational. What holds true for development at OneChronos does not hold true in general.

Agile's use of extensive task/resource based planning is a key driver of its adoption at companies that view software development as a cost center. Viewed through that lens agile gives the business transparency into where money is going and who to blame when projects go sideways. Most companies fall into the "software is a cost of doing business" camp, and the Agile Industrial Complex has grown strong as a result.

The use of project managers that are neither domain experts nor engineers writing code is something to avoid whenever possible. PMs are invaluable at companies where navigating the org structure or fighting for resources is itself a full time job. They're also invaluable when there's a culture of separating product owner roles and engineering roles to the point that engineers don't understand what the product does/why it exists and product owners don't have a clue how the sausage gets made.

When that's not the case, "project management for the sake of project management" is never a good thing. At best it's overhead. At worst it's another opportunity for introducing modeling errors as engineers and product owners play telephone through an intermediary.

OneChronos believes that the highest velocity product development possible comes when the domain experts themselves build systems, and having developers work directly with domain experts via a tight feedback loop is a fast second. We also believe that while system level planning is invaluable (ergo the strong emphasis on DDD), the value of time and resource based planning is dubious when stakeholders are economically aligned and on the "same team."

We believe in this strongly enough that it informs our decisions on hiring and corporate structure. Building product in this fashion requires a strong emphasis on keeping teams small, making every hire count, investing heavily in developing employees, and keeping incentives aligned. Almost every iconic company for which tech was the core differentiator started with this philosophy and worked to preserve it for as long as possible. Microsoft, Apple, Google, and Intel are notable examples.

## Closing Thoughts

We should live in a golden age for computers and software engineering. Cloud computing, automated devops, a growing emphasis on developer tooling, and increasingly sophisticated programming languages and frameworks have made it easier than ever to engineer software that's correct, performant, and easy to evolve.

Instead, by most measures, we live in a dark age. The Agile Industrial Complex, bloated frameworks, anemic mainstream programming languages, and culture that makes Silicon Valley the comedy seem like a documentary all contribute to that. The result is a software engineering process that's cumbersome, and software that's often underwhelming. Computers are capable of incredible feats of performance that most end users don't get to see. Instead, what most people see are systems that are [perceptually slower than they were forty years ago](https://twitter.com/gravislizard/status/927593460642615296).

Both the good and the bad parts of the software engineering status quo is great news for OneChronos. Our business will succeed on the basis of our ability to make and market software that seems like magic to end users. Embracing a unique, purpose driven approach to engineering is a big part of that.

[^validation_graph]: [Symbolic and Concolic
Execution of Programs. Omar Chowdhury 2015](https://www.cs.purdue.edu/homes/ninghui/courses/526_Fall15/handouts/SymExec.pdf)
