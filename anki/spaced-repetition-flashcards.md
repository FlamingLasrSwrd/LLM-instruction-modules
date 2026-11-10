# Spaced-Repetition Flashcard Best Practices (AI reference)

Synthesized from Gwern Branwen's "Spaced Repetition for Efficient Learning",
Piotr Wozniak's "Twenty Rules of Formulating Knowledge" (SuperMemo), and Jack
Kinsella's "Janki Method" and "Janki Method Refined" (links in §10). Use this
as the checklist whenever generating flashcards for Anki (for example through
an AnkiConnect-based MCP server): review each draft card against the rules
below before adding it. Tool names such as `add_flashcard`,
`add_flashcards_bulk` and `get_note_type_fields` refer to one such server;
substitute your own.

> **Last verified 2026-11-10** against the SuperMemo article, an archived copy
> of the Janki Method, search excerpts of "Janki Method Refined" (no live copy found), and the AnkiConnect
> listing on AnkiWeb.

## 1. Before writing any cards

- **Understand first, memorize second.** Don't turn material into cards until
  you (and the user) actually understand it. Cards for ununderstood material
  are near-worthless and create confident-sounding wrong answers later.
- **Get the big picture before the details.** If the user wants cards on a
  topic they haven't studied yet, prefer a handful of high-level cards over
  many detailed ones until the shape of the subject is established.
- **Prioritize.** Not everything is worth a card. Ask (or infer) what's
  actually important before ankifying everything a user pastes at you. Fewer,
  well-chosen cards beat exhaustive coverage.

## 2. Detail level

Pick one of three tiers before drafting cards. **Default to Skim** unless the
user names a tier, their request implies more ("go deeper", "also cover the
physiology"), or they've said they're studying for an exam or building
expertise in the subfield.

- **Skim** (default, ~5-8 cards per source): the core thesis and big-picture
  mechanism only. No named researchers/years, no specific numbers, no
  secondary or supporting findings — just the handful of ideas someone would
  need to explain the topic in one sentence each.
- **Study** (~12-18 cards): adds the main supporting mechanisms and the 2-4
  most important named findings/experiments, still one atomic fact per card.
- **Deep** (~25+ cards): adds specific numbers, named studies/years,
  secondary findings, and open questions/edge cases. Use this tier when the
  user explicitly asks for "deep"/"comprehensive" cards or says they're
  studying for an exam or mastering the subfield.

State which tier is being used when proposing cards, so the user can bump it
up or down before anything is added. Regardless of tier, still apply the
minimum information principle (below) — a lower tier means fewer, bigger-
picture facts, not compound cards.

## 3. The core rule: minimum information principle

Each card should test exactly one atomic fact, in the simplest form possible.
If a piece of knowledge has multiple parts, split it into multiple cards
rather than one card with a compound answer.

- Bad: "What are the causes, dates, and outcomes of the French Revolution?"
- Good: separate cards for one cause, one date, one outcome each.

Redundancy is fine and even encouraged — the same fact approached from two
angles (e.g., "What is X called?" and "Y is the term for what?") is not a
violation of this principle. Minimum information means simple *per card*, not
minimal total coverage.

## 4. Prefer active recall over recognition

- Write cards as free-recall/short-answer questions, not multiple choice.
  Recognizing an answer is weaker practice than retrieving it, and showing
  wrong options can leave them feeling familiar later. Anki's Front/Back
  format already suits recall; keep it that way rather than embedding
  answer options in the Front field.
- The Front should force genuine retrieval ("What does X do?"), not just
  pattern-match to something visible on the card.

## 5. Avoid these card shapes

- **Unordered sets** ("name the five..."). Wozniak (rule 9): it is "nearly
  impossible" to memorize sets of more than five members without mnemonics,
  enumeration or grouping. Either supply a mnemonic
  explicitly on the card, or split into individual cards with distinguishing
  context per item.
- **Ordered sequences / multi-step processes.** Don't make one card test an
  entire ordered list. Prefer overlapping cloze-style cards (each testing one
  step plus its neighbor) or a checklist the user reviews outside of Anki.
- **Ambiguous wording that causes interference.** If two cards' answers are
  easily confused (similar dates, similar names, similar function
  signatures), add distinguishing context to each card's Front so recall of
  one doesn't get tangled with the other.

## 6. Wording and formatting

- Optimize for speed of recognition once the answer is known: short, precise
  wording beats a complete grammatical sentence. Cut words that don't
  disambiguate the question.
- Add context cues (a category label, a source tag, a deck/subdeck) instead
  of restating context in every Front — this keeps individual cards terse.
- Use **bold** on the key term in the Back (via HTML in the field, e.g.
  `<b>term</b>`) so review is faster — the eye should land on the answer
  immediately, not have to parse a full sentence.
- Use cloze-style deletion when it's a more natural fit than Q/A (e.g.
  vocabulary in context, a formula with one missing term). If the note type
  in use is "Basic", simulate this by putting the cloze sentence in Front
  with a blank, and the missing term in Back — or ask the user whether to use
  Anki's real "Cloze" note type instead (`get_note_type_fields` first).
- Link new facts to things the user already knows when possible ("like X but
  for Y") — it shortens the wording needed and reduces interference.
- Personalize when it helps retention (the user's own examples, projects, or
  prior conversations) — personal/emotionally salient examples are recalled
  better than generic ones.

## 7. Provenance and volatility

- For facts that could go stale (library versions, API details, statistics,
  "current" anything), add a tag or a short date/version note in the card
  (e.g. tag `as-of-2026`) so the user can spot outdated cards later.
- For non-obvious or contestable facts, tag or note the source so the user
  can verify it later without re-researching from scratch.

## 8. Technical/programming material (Janki-style)

- Favor cards that test *what a function/concept is for* and *when to reach
  for it*, not exhaustive parameter lists — that's what docs are for.
- "Hybrid" cards (Kinsella's term) combine a summary of what a command or
  function is for with the details of how it operates, so the card can be
  pulled out and studied in depth when the feature is actually needed.
- Read real code and ankify interesting techniques (an algorithm, a hack,
  an architectural decision) with the code shown; also ankify your own past
  mistakes once you've worked out their cause.
- "What's great about this code?" / "What's wrong with this code?" cards
  (Front = a snippet, Back = the analysis) are effective for building taste,
  not just recall.
- Prefer concepts that transfer across languages/tools over
  framework-of-the-week trivia, unless the user specifically wants the
  latter.
- Skip ankifying fringe tech (languages, libraries, features the user
  rarely uses); make those cards only inside a project that actually uses
  it.

## 9. Workflow notes for Claude specifically

- When a user pastes a large chunk of material ("make flashcards on this
  chapter/doc"), don't mechanically generate one card per sentence. Pick a
  detail tier (section 2, default Skim), extract the atomic, important facts
  for that tier, apply the minimum-information principle, then propose the
  card list and count before bulk-adding.
- Default to the "Basic" note type (Front/Back) unless the user's content is
  clearly cloze-shaped or they ask for a different note type.
- When unsure whether something is worth a card at all, ask: "what real
  situation would you need to recall this in?" If there isn't one, don't
  make the card — suggest skipping it instead.

## 10. Sources

- Piotr Wozniak, [Twenty rules of formulating knowledge](https://www.supermemo.com/en/blog/twenty-rules-of-formulating-knowledge) (SuperMemo, 1999). Rules 1 to 20 map onto §1 and §3 to §7 above: understand first, minimum information, cloze deletion, avoid sets and enumerations, combat interference, optimize wording, context cues, redundancy, sources, date stamping, prioritize.
- Gwern Branwen, [Spaced Repetition for Efficient Learning](https://gwern.net/spaced-repetition) (2009, revised since): the research background, including the testing effect.
- Jack Kinsella, "Janki Method: Using SRS to Improve Programming" ([archived copy on gwern.net](https://gwern.net/doc/www/www.jackkinsella.ie/7f766c2f8b533e44f5a9ff02ea44573c3132557a.html)) and "Janki Method Refined". The original jackkinsella.ie domain now serves unrelated content; don't link to it. No live copy of "Refined" could be found on 2026-11-10 (the mirror that used to host it returns 404), so the hybrid-card, "what's great/wrong with this code" and fringe-tech points in §8 were checked only against search-result excerpts of it.
- [AnkiConnect](https://ankiweb.net/shared/info/2055492159) (add-on code `2055492159`; source at git.sr.ht/~foosoft/anki-connect): the local HTTP API that Anki MCP servers use.
