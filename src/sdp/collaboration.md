# Collaboration

Before discussing techniques for effective collaboration it's worth unpacking what we mean by stakeholder. Every company has internal and external stakeholders. Internal stakeholders include engineers, business development professionals, product owners, and the company itself. External stakeholders include customers, regulators, and vendors. Investors/non-employee shareholders lay somewhere in the middle.

Almost every initiative involves two or more stakeholders. Whenever that's the case, effective collaboration and a shared interest in the outcome is key to success. At times that collaboration is entirely commercial. Sometimes it's entirely technical. [DDD](../ddd/index.html) applies to both situations and everything in-between. Unlike most of the SDP religions, DDD is à la carte and has no methodological or technological underpinnings. Its core philosophy is simple:

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
