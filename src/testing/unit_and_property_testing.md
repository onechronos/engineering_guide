# Unit and Property Testing

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

stands a much better chance of catching said bugs on day one, and would prove less fragile in the face of future changes. NB: property testing and fuzzing \\( \neq \\) formal methods. They're random test case generation with (best case) some light static analysis to exercise interesting code paths. The effectiveness of property tests hinges on writing good "strategies" for generating test cases. Good property testing frameworks make this as easy as possible and far more declarative than cranking out unit tests, but they still require care to exercise corner cases.

As an experienced developer it's easy to think that you'd never miss trivial corner cases or make silly mistakes. The reality is, software is subtle and hard, and everyone misses corner cases. Fun fact: a bug in Java's implementation of binary search (written by the veritable Josh Bloch himself) went undetected for [over twenty years](https://en.wikipedia.org/wiki/Binary_search_algorithm#Implementation_issues).

Automated theorem provers and property testing frameworks are better at hitting corner cases than humans, but not because they're smarter. To the contrary, they're effective tools for finding bugs *because* they're dumb and pathological. Humans are smart enough to understand high level intent, but computers aren't. Bugs arise when humans make implicit assumptions and don't include those assumptions when writing down dumb, low level logic.

## Closing Notes on Unit Tests

When to unit test:

* Whenever you're working with a dynamic language
* When doing so helps your development process
* When property testing or formal methods are prohibitively difficult to apply to the problem at hand

When to avoid unit tests:

* The code in question is business logic - we'll get to testing business logic later
* Whenever what you're testing for is implicitly or explicitly checked by linting or compilation
* Whenever property testing could hit the corner cases that you're aware of, and then some

When unit tests find use, make sure to use them in conjunction with code coverage tools. The following is a list of the preferred unit testing and code coverage frameworks:

### Rust

* __Unit testing framework__: built-in
* __Code coverage:__ [Tarpaulin](https://github.com/xd009642/tarpaulin): Rust code coverage

### OCaml

* __Unit testing framework__: [alcotest](https://github.com/mirage/alcotest)
* __Code coverage__: [bisect_ppx](https://github.com/aantron/bisect_ppx)

### Elixir

* __Unit testing framework__: built-in
* __Code coverage__: [excoveralls](https://github.com/parroty/excoveralls)

### Python

* __Unit testing framework__: built-in
* __Code coverage__: [coverage.py](https://coverage.readthedocs.io/en/stable/)

### Java

* __Unit testing framework__: [JUnit](https://junit.org/junit5/)
  * __Mocking__: [Mockito](https://site.mockito.org/)
* __Code coverage__: [Clover](http://openclover.org/): Java code coverage
