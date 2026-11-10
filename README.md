# LLM instruction modules

Reference material for AI assistants working with tools and standards that
sit outside a model's training data, or that change faster than it. Each
module is plain Markdown that an assistant can load as context (for
example through a `CLAUDE.md` import or a skill).

| Module | File | What it covers |
|---|---|---|
| Structr | [`structr/structr-reference.md`](structr/structr-reference.md) | Building and changing a Structr (graph application platform) app through its REST API and scripts. Every statement is marked as verified on 6.0.0 or 7.1.0, present in 6.0.0, or taken from the 7.x documentation |
| BFO | [`bfo/bfo-cheatsheet.md`](bfo/bfo-cheatsheet.md) | Using Basic Formal Ontology to drive data modeling, especially in property graphs: taxonomy, relations, a classification procedure, modeling patterns, pitfalls |
| BFO | [`bfo/bfo-2020-reference.md`](bfo/bfo-2020-reference.md) | Lookup table of BFO 2020 classes and relations with declared domains and ranges, plus the IAO, CCO and RO terms used beside them |
| Anki | [`anki/spaced-repetition-flashcards.md`](anki/spaced-repetition-flashcards.md) | Rules for writing good spaced-repetition flashcards, for an assistant that creates Anki cards |
| Home Assistant | [`home-assistant/best-practices/`](home-assistant/best-practices/SKILL.md) | The community `home-assistant-best-practices` skill, vendored unmodified (MIT, see below) |
| Home Assistant | [`home-assistant/agent-field-notes.md`](home-assistant/agent-field-notes.md) | API mechanics and traps from managing a real instance over REST and WebSocket, generalized |

## Verification

Every module was checked against live sources on **2026-11-10**; each file
opens with a "Last verified" note naming what it was checked against and
what could not be checked. In short:

- **Structr:** setting names and defaults and script function names against
  Structr's source (`main`, 7.2-SNAPSHOT); 7.1 items against the 7.1.0
  release notes; a sample of documentation-sourced items against
  structr.org/docs. Empirical findings were not re-run.
- **BFO:** against the BFO-2020 repository's terms and relations tables and
  OWL files, IAO, RO, CCO, Crossref and the cited papers.
- **Anki:** against Wozniak's "Twenty rules", an archived copy of the Janki
  Method, and AnkiWeb.
- **Home Assistant:** the best-practices skill is upstream `main` at commit
  `166defa` (2026-10-29, skill version 56); the field notes against HA core
  source, developer docs, integration docs and js-yaml.

These tools move quickly. Treat anything version-specific as a starting
point and check it against your own instance.

## Licensing

`home-assistant/best-practices/` is copied from
[homeassistant-ai/skills](https://github.com/homeassistant-ai/skills)
(`skills/home-assistant-best-practices/`, commit `166defa`) under the MIT
License; its [`LICENSE`](home-assistant/best-practices/LICENSE) file applies
to that directory. To update it, copy the directory from upstream again
rather than editing it here.

Everything else is released under the MIT License in [`LICENSE`](LICENSE).
