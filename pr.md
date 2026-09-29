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

Description: Updates `classic-thesis-uct` from 0.1.3 to 0.1.4.

- Adds `side-caption-algorithm` and `list-of-algorithms-page` for numbered, side-captioned pseudocode blocks, with the algorithm counter reset per chapter like figures and tables.
- Fixes strong/bold text: the `EB Garamond 12` body face ships no bold, so `*strong*` silently rendered regular. It now falls back to the bundled `EB Garamond` bold face (Adobe Garamond Pro keeps its own).
- Fixes small caps: `smcp` is now requested for both font choices, so EB Garamond headings and `spaced-caps` render in small capitals instead of plain text.
- Splits the title-back name, title and date onto separate lines.
- Documents the new `EB Garamond` bold dependency and the algorithm functions in the README.
