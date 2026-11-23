# Domain documentation

- Use `wiki/` for knowledge about microcontrollers and embedded development. Check related pages before adding a new one.
- Start each numbered wiki chapter's filename with the number in its main heading, as in `1-hardware-platforms.md`.
- Use `docs/` for the repository's structure, conventions, and workflows.
- Keep shared domain terminology in `wiki/GLOSSARY.md` when a glossary is needed. Use its terms consistently across wiki pages.
- Keep decisions about repository structure and editorial conventions in `docs/adr/`. Consult relevant records when changing those conventions.

## Keep the README contents in sync

Use the numbered headings in `wiki/` as the source for the `Зміст` section in `README.md`. List the chapters and their numbered subsections in numerical order and match the nesting in the wiki.

- **Add** a chapter or subsection by inserting its entry in the matching position. Link chapters to their files and subsections to their headings.
- **Edit or move** a chapter or subsection by updating its entry wherever the title, number, location, or nesting changes. Update the link target when the file path or heading changes. Rename the chapter file if its chapter number changes.
- **Delete** a chapter or subsection by removing its entry and any entries for subsections removed with it.

Before finishing any wiki change, check that every current numbered heading has exactly one matching entry and that each entry links to the correct file or heading. Check this after text-only edits as well.
