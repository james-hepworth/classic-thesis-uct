<!--
Thanks for submitting a package! Please read and follow the submission guidelines detailed in the repository's README and check the boxes below. Please name your PR as `name:version` of the submitted package.

If you want to make a PR for something other than a package submission, just delete all this and make a plain PR.
-->

I am submitting
- [ ] a new package
- [x] an update for a package

<!--
Please add a brief description of your package below and explain why you think it is useful to others. If this is an update, please briefly say what changed.
-->

Description: Updates `classic-thesis-uct` from 0.1.4 to 0.1.5.

- `side-caption-algorithm` is now a `figure` of `kind: "algorithm"`, so algorithms can take a label and be cited with `@` like figures and tables (e.g. `#side-caption-algorithm(...) <alg:foo>` then `@alg:foo` renders "Algorithm 4.1"). Previously it returned plain content, so a label had nothing to attach to.
- Numbering now uses the figure counter for that kind, still reset per chapter, and the List of Algorithms is unchanged.
- The template's example algorithm in chapter 4 now carries a label.
