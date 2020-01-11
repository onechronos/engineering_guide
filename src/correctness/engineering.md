# Engineering for Correctness

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
