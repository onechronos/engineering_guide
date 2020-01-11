# Fault Types

* __Type I__: faults that result from programming and operational errors of the type that are reasonably knowable and avoidable. Examples of faults in this category include:
  * Straight up programming bugs, e.g. de-referencing a null pointer, introducing data race conditions, invalid casts, overflow/underflow, etc
  * Deploying software incorrectly
  * Provisioning or configuring hardware or backbone infrastructure e.g. DNS servers incorrectly
* __Type II__: faults that result from failures in software/hardware made or operated by parties external to the organization. Examples include:
  * Crashing after calling into an API function exposed by a third party library
  * Data corruption resulting from buggy device drivers or hardware operating out of spec
* __Type III__: faults that result from a system functioning as-implemented, but the implementation itself failing to capture either the higher level intent of the business or the programmer. Examples of Type III faults include:
  * Implementing a mathematical algorithm in a way that's faithful to the specification of the algorithm, but the algorithm itself being wrong for the task at hand
  * Implementing business logic in a way that's faithful to a specification, but not in line with domain expert intent

We'll defer the discussion of Type I faults to a later section. DDD *is* an effective tool for dealing with them - but not the most effective one. Type I faults are faults introduced by humans making mistakes, so anything that makes humans less error prone reduces the probability of Type I faults. Reducing cognitive load is widely known to make humans less error prone, and DDD reduces cognitive load. That said, humans will always be more error prone than automated, deterministic processes. As such, the best tools for fighting Type I errors are the automated ones that deserve a full treatment independent of DDD.

The ability of DDD to prevent Type II faults isn't obvious, but as it turns out, DDD is one of the best tools available for avoiding them. As for why, DDD pushes organizations towards architectures with explicit failure modes, isolation, redundancy, and recovery capabilities. That's a good thing in general, but crucial to ensuring the reliability of systems with unknown unknowns; the failure modes of external dependencies are often unknown unknowns.

Type III faults are often the most challenging and business impactful. Type I and II faults have to do with failures of software. Type III faults result from failures of the software development process, and almost always result in business impactful failures.
