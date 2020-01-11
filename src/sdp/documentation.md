# Documentation

In general, you should document systems, not code. Line level code comments for the sake of code comments aren't valuable. More often than not well named variables and type signatures are enough. When they're not, that's often cause for concern; rote code should be readable without commentary.

In general, comment code if one of the following is true:

* You're documenting an algorithm or something that's unreadable for a good reason, e.g. "The following is a trick for computing $x \text{ mod } n$ without division. It works because $n = 2^k, k \in \mathbb{Z}$. See [Compute modulus division by 1 << s without a division operator](https://graphics.stanford.edu/~seander/bithacks.html#ModulusDivisionEasy)."
* You're documenting a __stable__ API that will __definitely__ find use outside of the project that it lives in. Note the emphasis on "stable" and "definitely."
* You're documenting design decisions. These notes are often invaluable for understanding code and preventing future teams (or yourself) from wasting time on pursuing dead ends when revisiting code down the road.

Code comments aside, systems should come with extensive and actively maintained documentation. Every code project should contain a README explaining (at minimum):

* What the project does
* How to build, run, and deploy the project
* What the project's system level dependencies are
