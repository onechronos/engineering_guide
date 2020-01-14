# Software Development = Software Engineering

Writing code might or might not be *engineering.* Programming is and often should be an exercise in pragmatism - getting the task at hand done. That's fine for environments where the value of feature velocity outweighs the value of correctness - something that's true for most companies. That's not true for settings like OneChronos where mistakes are potentially catastrophic. [What happened at Knight](https://www.henricodolfing.com/2019/06/project-failure-case-study-knight-capital.html) is a cautionary tale. One should never *write code* in such environments; teams should work together to *engineer systems.*

The core differentiators between coding and engineering software are *planning*, *process* and *repeatability.* Planning is about figuring out what you want a system to do before you start implementing it. *Process* is all about making sure that all software gets built and deployed in a consistent fashion. Planning + process = repeatability.

Notably absent in this description of engineering is any prescribed formula for planning, or any fixed process. There's a reason for that. No development methodology is "one size fits all," even within an organization. In fact, given our strong emphasis on individual contributors the team is often one person working on a specific system in isolation. It doesn't matter how that person chooses to run their own development process, as long as they treat it like one and hit quality standards. When individuals team up for a project, they're free to collaborate as they see fit.

Given that teams are free to do as they please OneChronos needs a "meta process" to ensure uniform quality, interoperability, and readability. Keeping in line with our approach to everything else, the process is/will be mostly automated, and puts great trust in individual contributors.

__The following are the hard and fast firm-wide rules for software engineering at OneChronos:__

* All code must be tested and verified to the standards outlined in this guide's [section on testing](#testing).
* Take a DDD approach to designing and building systems.
* Take a polyglot approach to development and build systems in whatever language is the best fit. You can have a favorite language, but that should never be a factor in this decision.
* Favor safer and more performant languages whenever possible, but use the best tool for the job. For example, python is a slow and "dangerous" language, but way better suited for research tasks than OCaml or rust.
* All compiled code must compile warning-free with all warnings enabled.
* Use "low cost" static analysis tools when available.
* Automate everything involving build / test / deployment process.
* Don't bikeshed. That applies 10x when it comes to syntax and matters of personal taste. The OneChronos style guides might not make sense in every situation but the benefits of uniform use are twofold:
  * Code that appears uniform company wide is easier to read \\( \Rightarrow \\) reduced cognitive load \\( \Rightarrow P(\text{mistake}) \\)
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
