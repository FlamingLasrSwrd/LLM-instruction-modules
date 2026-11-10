# Basic Formal Ontology (BFO) Cheat Sheet

Project-agnostic notes on using [Basic Formal Ontology](https://basic-formal-ontology.org/) (a
top-level/foundational ontology, now an ISO/IEC standard) to drive data
modeling for real applications — especially graph-database-backed ones.
Written for reuse across unrelated projects, the same way you'd reuse a
language cheat sheet.

**Sources this draws on** (see §9 for full links):
- Arp, Smith & Spear, *Building Ontologies with Basic Formal Ontology*
  (MIT Press, 2015) — the standard textbook; cited below as "the book."
- The official BFO 2020 GitHub repo, conformant to ISO/IEC 21838-1/-2 —
  including the current, authoritative **terms table** (122 rows with
  formal elucidations/definitions/examples/domain/range) and **relations
  table** (45 rows: every core relation plus its inverse and temporalized
  variants), both pulled directly from the repo's own spreadsheets.
- Hands-on lessons from actually building a BFO-informed graph-database
  schema for a real application (an operations tracker for a plant
  nursery, on Structr/Neo4j) — marked **🔧 Field-tested** below where the
  lesson came from real friction, not just reading.

> **Last verified 2026-11-10** against BFO-2020 `master` (commit `bd1fb61`,
> 2026-10-06): terms/relations tables, IAO `iao-edit.owl`, RO `ro-edit.owl`,
> the PMC RC-account paper, Crossref for the *Ratio* citation, and every
> link in §9. Claims sourced only to the 2015 book (Box numbers, the 25
> principles) were not re-checked against the book's text.

---

## 1. What BFO is, in one paragraph

BFO is a *domain-neutral* ontology: a small, fixed set of the most
general categories of thing that can exist (`object`, `process`,
`quality`, `role`, ...), plus the relations between them (`part_of`,
`participates_in`, `has_quality`, ...). It doesn't know what a "cell" or
a "customer" or a "greenhouse" is — that's your domain's job. What it
gives you is a place to *hang* your domain's types so that (a) you don't
have to reinvent categories like "is this a thing, an event, or a
property of a thing?" from scratch, and (b) your ontology stays
interoperable with everyone else's, since it's the shared backbone
behind the entire [OBO Foundry](https://obofoundry.org/) (Gene Ontology,
Cell Ontology, Protein Ontology, Foundational Model of Anatomy, and
~150+ others) and is now an ISO/IEC standard (21838-1 for top-level
ontologies generally, 21838-2 for BFO specifically).

**Why bother for an application (not a research ontology)?** Even
outside biomedicine, BFO's category split answers questions your schema
will otherwise get wrong by accident:
- Is this field a *thing* (make it a node/entity), an *event* (make it a
  node with a time), or a *property* (make it a field or a relationship
  to a reference type)? — The single most common real-world modeling
  bug is treating a participant-in-an-event as a free-text description
  field instead of a real relationship (see §7.3).
- Does this "thing" actually change what kind of thing it is over its
  life (a cutting becomes its own plant; a batch gets divided)? BFO has
  an explicit, narrow answer for exactly this (§2.1).
- Is "for sale," "customer," "admin" a permanent fact about an entity,
  or an assignable, revocable status? BFO's Role category exists
  precisely to stop you from modeling a role as if it were a rigid type
  (§2.3).

## 2. The big split: Continuant vs. Occurrent

Everything BFO classifies is either a **continuant** (persists, endures,
exists "in full" at any moment it exists at all — a person, a rock, the
color of a tomato) or an **occurrent** (unfolds in time, has temporal
parts, never exists "as a whole" at any single instant — a process, a
war, a heartbeat). This is *the* first question to ask about anything
you're modeling. Neither view reduces to the other — BFO deliberately
holds both perspectives at once (the book calls this "perspectivalism"):
you can describe an organism's *anatomy* (continuant slice) or its
*physiology* (occurrent unfolding) and both are equally real
descriptions of the same reality, just at different "grain" and from
different angles.

Practical tell: if you can ask "what does it look like *right now*?"
and get a complete answer, it's a continuant. If the only complete
answer requires the whole history from start to finish, it's an
occurrent. A "surgery" isn't complete at any instant during it — it's
an occurrent. The patient is a continuant throughout.

## 3. Continuant taxonomy (cheat table)

| Category | Elucidation (BFO 2020 official wording, condensed) | Examples | Graph-modeling note |
|---|---|---|---|
| **Independent continuant** | Bears qualities; doesn't itself depend on/inhere in anything else | an organism, a chair, a spatial region | The "real things" — usually your main node types |
| → **Material entity** | Independent continuant with some portion of matter as part | a human, an undetached arm, an aggregate of humans | Splits into the three below |
| → → **Object** | Material entity that manifests causal unity and is maximal relative to that kind of causal unity | an organism, a fish tank, a planet, a laptop, an ice cube | Your typical "one real thing" node |
| → → **Object aggregate** | Material entity consisting exactly of a plurality of objects as member parts which together form a unit | the musicians in a symphony orchestra, the nitrogen atoms in the atmosphere, a collection of cells in a biobank | A grouping node with real membership — not just a tag |
| → → **Fiat object part** | Proper part of an object, demarcated from the rest of it by one or more fiat surfaces (not a physical discontinuity) | the upper and lower lobes of the left lung, the Western hemisphere of the Earth | Usually **doesn't need its own node** if it's only ever referenced within one event (see §7.4) |
| → **Immaterial entity** | Independent continuant with no material entity as part | a boundary, a site, a spatial region | |
| → → **Site** | 3-D immaterial entity whose boundaries coincide (wholly/partly) with those of material entities, or are located relative to some material entity | a rabbit hole, the lumen of your gut, the hold of a ship | Model *location* as a Site, not as a flat string — 🔧 field-tested: this is the single highest-leverage upgrade over a naive "location: string" field |
| → → **Continuant fiat boundary** | Zero/one/two-dimensional immaterial entity, with no spatial region as part, whose location is determined relative to some material entity | the Equator, the North Pole, the surface of a cell membrane | Rarely needs modeling explicitly unless boundary-adjacency itself is a real query |
| → → **Spatial region** | Part of space itself, not dependent on any material entity, doesn't move | (0D) a point, (1D) a line, (2D) a surface, (3D) a volume | Only worth modeling explicitly for GIS-style apps |
| **Specifically dependent continuant** | Depends on exactly one specific bearer for its existence — can't migrate to another bearer | a color, a mass, a role, a disposition | |
| → **Quality** | Depends on a bearer, fully realized/exhibited (no further process needed) | the color of this tomato, the mass of this kidney | Model as a *timestamped instance*, not an overwritten field, if it changes (§7.1) |
| → → **Relational quality** | A quality with more than one bearer | a marriage bond, an instance of love, an obligation between one person and another | |
| → **Realizable entity** | Depends on a bearer, but is only *exhibited* through characteristic processes it can participate in | a role, a disposition, a function | |
| → → **Role** | Externally grounded — exists because of circumstances the bearer doesn't have to be in; optional; losing it doesn't change the bearer's physical makeup | the customer role, the doctor role, "acting as collateral" | Model as a wrapper/assignment node the bearer can gain and lose (or, if the role has no state of its own, a bare relationship on the realizing process instead) — 🔧 field-tested pattern, §7.2 |
| → → **Disposition** | Internally grounded — exists because of the bearer's physical makeup; losing it *does* change the bearer physically | an atom's disposition to decay, a person's predisposition to colon cancer | |
| → → → **Function** | A disposition that exists because the bearer was selected/evolved/designed to perform it | a heart's function to pump blood, a hammer's function to drive nails | |
| **Generically dependent continuant** | Depends on *some* independent continuant as bearer, but *can migrate* between bearers (via copying) | a PDF file, a novel's text, a musical score, a DNA sequence pattern | Information-content-entity territory — a recipe/plan/specification is this category too, see §7.11 |

**The one rigidity exception** (worth memorizing — it's the single
weirdest/most-useful fact in BFO): every other BFO category is *rigid*
— an instance can never stop being an instance of it without ceasing to
exist. `Object`, `Object aggregate`, and `Fiat object part` are the
**only** three categories a single continuant can migrate between over
its life. A cutting is a fiat object part of its mother plant until
severed, then becomes its own object. A batch of seedlings might be
tracked as an aggregate, then split into individually-tracked objects.
Plan your schema for this transition (a real node that changes "kind,"
or a controlled split-into-new-node event) rather than being surprised
by it. See the book's chapter 5 (continuants) and §7.5 below.

## 4. Occurrent taxonomy (cheat table)

| Category | Elucidation | Examples |
|---|---|---|
| **Process** | Occurs/happens, has temporal parts, depends on at least one material entity as participant | a heartbeat, a surgery, a chemical reaction, an order being placed |
| → **History** | The *sum of all* processes taking place in the spatiotemporal region occupied by the material part of a material entity — each material entity has exactly one (`history_of`) | "the life of this organism" |
| **Process boundary** | A temporal part of a process that has no proper temporal parts (an instantaneous edge) | the boundary between the 2nd and 3rd year of your life, the moment of birth |
| **Spatiotemporal region** | The 4-D region (space + time) a process occupies | the region occupied by a war |
| **Temporal region** | The purely temporal projection of the above | (0D) an instant, (1D) an interval |

Processes are BFO's natural **event/log-entry** category — in an
application schema, your event/transaction/log tables (`SowEvent`,
`OrderPlaced`, `TreatmentApplied`, ...) are Processes. This is also
BFO's built-in answer to n-ary/reified relations — see §7.3.

**"Process Profile" is not a current BFO category — don't build around
it.** Proposed by Smith for BFO 2.0 (*Classifying Processes: An Essay in
Applied Ontology*, *Ratio* 25:4, 2012) to represent a *measurable
dimension of an already-occurring process* — the worked example is a
heartbeat: "a rate process profile is an occurrent that is an
occurrent-part-of some process by virtue of the rate of change in an
attribute of one or more participants of that process" (a rhythm
process profile similarly captures variation in that rate over time).
It's about **measurement of a process that's already happening**, not
about representing one that hasn't happened yet — and multiple sources
confirm it was ultimately left out of the ratified BFO spec entirely. If
you're looking for how to model a *future/planned* process instead
(scheduling, not measurement), that's a different question with a
different, current answer — see §7.11.

## 5. Core relations (from the official BFO 2020 relations table)

Every relation below ships with a formal inverse, and (for BFO 2020-OWL)
temporalized variants ("at all times" / "at some time" — because
continuant relations can hold only intermittently, e.g. a kidney is
only `located_in` a torso while it hasn't been transplanted out). Full
definitions/examples are in the [official terms
table](https://github.com/BFO-ontology/BFO-2020/blob/master/21838-2/owl/profiles/temporal%20extensions/temporalized%20relations/documentation/bfo-2020-terms.xlsx).

| Relation | Inverse | Holds between |
|---|---|---|
| `is_a` | — | universal ↔ universal (logical primitive; not in the relations table) |
| `instance_of` | — | particular → universal, at a time (logical primitive; not in the relations table) |
| `continuant_part_of` | `has_continuant_part` | continuant ↔ continuant, at a time |
| `occurrent_part_of` | `has_occurrent_part` | occurrent ↔ occurrent (atemporal) |
| `member_part_of` | `has_member_part` | object → object aggregate |
| `located_in` | `has_location` | independent continuant ↔ independent continuant (neither a spatial region) |
| `inheres_in` | `bearer_of` | specifically dependent continuant → independent continuant |
| `generically_depends_on` | `is_carrier_of` | generically dependent continuant → independent continuant |
| `concretizes` | `is_concretized_by` | specifically dependent continuant/process → generically dependent continuant |
| `has_participant` | `participates_in` | process → continuant |
| `realizes` | `has_realization` | process → realizable entity (role/disposition/function) |
| `has_material_basis` | `material_basis_of` | disposition → material entity |
| `preceded_by` | `precedes` | occurrent ↔ occurrent |
| `occurs_in` | `environs` | process → material/immaterial entity it happens inside |
| `history_of` | `has_history` | history → material entity |
| `specifically_depends_on` | `specifically_depended_on_by` | specifically dependent continuant → independent continuant (not a spatial region) or another specifically dependent continuant |

The relations table also has the `proper_` variants of the part
relations, `temporal_part_of`, `first_instant_of`/`last_instant_of`,
`occupies_{spatial,temporal,spatiotemporal}_region`, and
`spatially_/temporally_projects_onto`.

⚠️ **Not BFO**: `derives_from` and `adjacent_to` do not appear in either
official BFO 2020 spreadsheet (terms table: 122 rows; relations table:
45). `derives from` lives in the OBO Relation Ontology (RO_0001000), and
RO flags it as an older relation that may be replaced by its
"develops from" family. Import it from RO if you need it; don't present
it as BFO.

**Formal properties** (from the book's Box 7.3 — check these before you
assume a relation you're building behaves like one you already know):

| Relation | Transitive | Symmetric | Reflexive | Antisymmetric |
|---|---|---|---|---|
| `is_a` | ✓ | – | ✓ | ✓ |
| `part_of` | ✓ | – | ✓ | ✓ |
| `located_in` | ✓ | – | ✓ | – |
| `preceded_by` | ✓ | – | – | – |
| `has_participant` | – | – | – | – |

**The all-some rule**: every universal-to-universal relation in BFO
(except `is_a` itself) means "**all** instances of the left type stand
in this relation to **some** instance of the right type" — never "all
to all." `human has_continuant_part testis` would fail this test (not
all humans have testes); `human testis continuant_part_of human` passes
it. When you're defining a relation between two of your own domain
types, run this check — it's the single fastest way to catch a
too-strong axiom before it produces a nonsensical inference later.

**Inverse ≠ reciprocal.** The literal inverse of a relation always holds
when the forward one does (`has_continuant_part` when `continuant_part_of`
holds). But an *informally* "opposite-sounding" relation often isn't the
true inverse and fails the all-some test — `human testis
continuant_part_of human` holds, but `human has_continuant_part human
testis` does not (not all humans have one). Don't assume a plausible-
sounding reverse relation is safe to assert both ways.

## 6. A classification decision procedure

When you hit a new field or concept in your domain and don't know how
to model it, walk this in order:

1. **Does it persist through time, or unfold in time?** → Continuant or
   Occurrent (§2). If you're not sure, ask "can I fully describe it at
   a single instant?" — yes → continuant.
2. **(Continuant) Does it depend on a bearer to exist, or stand on its
   own?** Stands on its own → Independent continuant. Depends on
   exactly one bearer, can't migrate → Specifically dependent
   continuant. Depends on some bearer but *can* migrate (via copying) →
   Generically dependent continuant.
3. **(Independent continuant) Is it made of matter?** No → Immaterial
   entity (a site, a boundary, a spatial region). Yes → Material
   entity, then: is it a single bounded, causally-unified whole
   (Object)? A plurality of objects forming a unit (Object aggregate)?
   Or a part of a larger object with no physical boundary of its own
   (Fiat object part)?
4. **(Specifically dependent continuant) Is it exhibited only through
   a process, or fully present without one?** Fully present → Quality.
   Needs a process to be exhibited → Realizable entity, then: does
   losing it change the bearer's physical makeup? No → Role. Yes →
   Disposition (and if the bearer exists *because* it was
   designed/evolved for that disposition → Function).
5. **(Occurrent) Is it the whole unfolding thing, or its instantaneous
   edge?** Whole thing → Process. Edge → Process boundary.
6. **Once you know the category, is it a *relation to something else*
   (make it a graph edge / relationship type) or an *intrinsic value*
   (make it a node property)?** Rule of thumb from the book (ch. 2):
   nouns → nodes, adjectives/single-values → properties, verbs/actions/
   facts-about-two-things → relations.

## 7. Graph-database modeling patterns

The book's own worked examples are all OWL/Protégé (description
logic, class-based). None of that changes the *categories* — it changes
how you'd implement them in a **property graph** (Neo4j, Structr, etc.)
instead. These are the translation patterns that came out of actually
doing it.

### 7.1 Qualities that change: snapshot + history, not an overwritten field
A Quality (§3) is a *specifically dependent continuant tied to one
instant's state* — but most schemas need both "what's the current
value" (fast reads) and "what was it at time T" (audit/traceability).
Don't overwrite a single field. Instead: append a new timestamped
Quality-instance node/event every time it changes, **and** materialize
the current value as a cached field on the bearer, written only when
the incoming timestamp is newer than what's already cached. 🔧
Field-tested: the naive version of this bug is subtle — if the
"is this newer?" check reads the cached field with a query that
silently returns only default/id fields (common in REST layers with
tiered "views"), the guard never fires and events silently apply
out of order. Verify the guard actually rejects an out-of-order write
before trusting it, don't just verify the happy path.

### 7.2 Roles as wrapper/assignment nodes, not boolean flags
Don't model "is this for sale" or "is this a customer" as a boolean or
enum on the underlying object — that's treating an externally-grounded,
optional Role (§3) as if it were a rigid, permanent fact. Model the role
as its own node that points at the bearer, can be created, paused, and
archived independent of the bearer's own lifecycle, and is queried
*through* rather than *on* the bearer. This also naturally gives you a
history of when something entered/left the role, for free.

**Two different implementations, pick by whether the role itself carries
state.** Not every Role needs a reified node — the deciding question is
whether the role has its own lifecycle worth tracking independently:
- **Reify it as its own node** (the pattern above) when the role has
  real state of its own — a status, a price, a start/end, something that
  changes independently of whether the bearer itself changes. "For sale"
  is a good example: it has a price and an active/paused/sold status
  that clearly belong to the *listing*, not to the underlying object.
- **Represent it as a bare relationship, no node at all**, when the role
  is fully exhausted by "which particular thing was the bearer, in which
  specific circumstance" — nothing else about the role itself needs
  tracking. 🔧 Field-tested: "pest" (an insect is only a pest *relative
  to* damaging some other organism, never an inherent property of the
  insect) needed no node of its own — the role is entirely captured by a
  single relationship on the specific event where the harm actually
  occurred (`StatusEvent.causedBy → the causing organism`). No property
  named "isPest" exists anywhere, on either party; the role is only ever
  evidenced relationally, and only for the specific process instance
  that realized it. Don't build the heavier reified-node pattern by
  default if a bare relationship already says everything there is to
  say.

Either way, the same underlying discipline holds: the Role is never a
stored property of the bearer itself. The two versions differ only in
whether the role also needs a home for its *own* data.

### 7.3 Processes as your reification mechanism for n-ary/qualified relations
Property graphs (and RDF) are naturally binary: an edge connects
exactly two nodes. Real events are rarely binary — "who did what to
what, when, using which materials, at what location" is n-ary. BFO
already gives you the right reification point: **model the event as its
own Process node**, and give every participant its own `has_participant`
/ `participates_in` edge to that one node, rather than trying to cram
multiple participants into properties on a single edge or bloating one
node with a dozen foreign-key-shaped string fields. This is the same
move the W3C's own [n-ary relations
note](https://www.w3.org/TR/swbp-n-aryRelations/) recommends generically
("use an intermediate node so the relation instance itself can be
annotated") — BFO just gives the intermediate node a principled meaning
(it's not an arbitrary reification hack, it's a real Process). 🔧
Field-tested: the fastest way to find every place your schema is
missing this is to audit every "free text description" field on an
event type and ask "is this actually naming a *participant* that should
be a real relationship instead of a string?" — that was, in one real
project, a 15-field gap discovered only by a systematic field-by-field
audit against exactly this question.

### 7.4 Fiat parts consumed within one process don't need a node
Not every Fiat object part (§3) needs to be a persistent node. "3 grams
of fertilizer applied in this treatment" is a fiat-demarcated portion of
a bulk substance that ceases to be distinct once consumed — representing
it as `amount` + `unit` properties directly on the consuming Process
node is the right level of granularity, not a fabricated node for a
part that never outlives the event. Reach for a real node only when the
part needs to be referenced *again* later (e.g., a cutting that is
severed and becomes tracked in its own right — that's the rigidity
exception in §3, not this case).

### 7.5 Plan for the rigidity exception explicitly
Since Object / Object aggregate / Fiat object part is the *only*
category triple a continuant can migrate across (§3), decide up front
how your schema represents that transition: a controlled "split" event
that creates a brand-new node and links it back to its origin (e.g.
RO's `derives_from`, which is not a BFO relation; see §5), rather than
mutating a type field on an existing node in place.
Silently changing a node's "kind" in place tends to break every query
and relationship that assumed a stable type.

### 7.6 Self-referential trees: keep Sites and Universal-hierarchies as two different trees, with two different relation labels
Two very different things want the same tree *shape* (a self-referential
parent pointer) and it's easy to conflate them into one generic
`PART_OF`-labeled relationship, because in a property graph both just
look like "a node pointing at its parent":
- A **Site hierarchy** (bench part_of bay part_of greenhouse) is a tree
  of *particulars* — real places — and the relation genuinely is
  mereological: a bench really is spatially part of a greenhouse. Label
  it `PART_OF` (or BFO's exact `continuant_part_of`).
- A **taxonomy/universal hierarchy** (species is_a genus is_a family) is
  a tree of *types*, built once as reference data — and the relation is
  **not** parthood. A species being a more specific rank than its genus
  is subsumption between universals (BFO's `is_a` — "Homo sapiens is_a
  Homo," the standard example), the exact same relation that structures
  an ordinary `is_a` taxonomy/class hierarchy. Label it `IS_A`, not
  `PART_OF`.

🔧 Field-tested: this is not a hypothetical mistake. A real schema
originally labeled its taxonomic rank hierarchy's self-relationship
`PART_OF` — an easy slip, since it has the identical self-referential
shape as a real, correctly-labeled `PART_OF` Site hierarchy sitting right
next to it in the same schema. Caught only on a deliberate second pass
auditing every relationship for its *category*, not its shape, and fixed
by renaming to `IS_A`. The fix itself carried its own gotcha on the
implementation side: in at least one graph platform, renaming a
relationship's label orphans every existing edge under it (the same way
renaming a node type can orphan existing instances) — so the rename
needs a real read-old/recreate-new/rewrite migration, not just an
in-place label swap. Check whether your own platform's relationship
renames are safe before assuming so.

Never let a real Entry/instance attach directly into the universal tree
as if it were a node in it — an instance *instantiates* a leaf universal
(e.g., a `PlantVariety`), which is `is_a` further up the universal tree.
Keep `instance_of` and `is_a`/`part_of` as genuinely different relations,
even though both "look like" parent pointers in a property graph.
Conflating them is the single most common way an otherwise-correct
self-referential-tree pattern goes wrong.

### 7.7 The "exactly one of N possible targets" pattern
A Role-wrapper or event often needs to point at *exactly one* of several
possible target types (a Listing is for sale as an Entry, a Group, a
Product, *or* a SeedLot — never more than one). Most graph databases
have no schema-level "exactly one of" constraint across heterogeneous
target types. Reuse one relationship label across all the possible
(source, target) type pairs (don't invent `LISTED_AS_ENTRY`,
`LISTED_AS_GROUP`, etc. — one `LISTED_AS` label reused, same as BFO
itself reuses e.g. `located_in` across every independent-continuant
pair) and enforce the "exactly one" constraint in application code at
write time, documented clearly since the schema itself won't catch a
violation.

### 7.8 Don't build structure ahead of a real question
BFO gives you *permission* to model fertilizer-as-tracked-lot, a full
Site hierarchy, a real genus/family taxonomy, individual-pot tracking,
roles for every conceivable party — but the ontology being *correct*
for a category of thing doesn't mean building it is warranted yet. Build
the category the moment a real question needs it ("how much fertilizer
is left," "show me everything in Greenhouse 2"), not speculatively
because BFO has a slot for it. An adequate ontology and a *populated*
one are different commitments — the book's own "principle of low-hanging
fruit" (ch. 3) says start with what's easiest and clearest and only
build outward as real need appears.

### 7.9 Verify your platform's inheritance mechanism before designing around it
If your graph platform supports type inheritance (a shared abstract
supertype for e.g. every event type's common fields, or every physical-
object type's shared relationships), **test it directly on your actual
instance before committing a schema design to it** — inheritance
mechanisms vary a lot between platforms and versions (single-inheritance
Java-style `extends`, multi-inheritance "traits," RDFS `subClassOf`, OWL
class axioms), and it is entirely possible to test the wrong mechanism,
get a false negative, and wrongly conclude the platform "doesn't support
inheritance" when it does — just under a different field/keyword than
the one you tried. Read a real, already-existing type definition from
your own instance to see which mechanism it actually uses, rather than
assuming based on a different platform or an older version's docs.

🔧 Field-tested, a second layer of the same lesson: even after finding
the *correct* mechanism, a polymorphic relationship (one edge type that
accepts any subtype of an abstract supertype as its target — the natural
implementation of §7.6's "reused label across every reference-kind"
pattern) can behave differently for **instances that already existed
before the type gained the trait** versus brand-new ones. In one
platform, a graph node's underlying type label is fixed at creation
time and isn't retroactively backfilled when its schema type is later
given a new supertype — so a relationship polymorphically targeting the
abstract type accepted a freshly-created instance immediately but
rejected every already-existing instance of the identical type,
including ones that inherited the trait *directly* (not just through a
multi-level chain — ruling out "chain depth" as the cause). A container
restart didn't fix it; touching/re-saving the old instance didn't
either. If you're retrofitting an abstract supertype onto a type that
already has live data, test the polymorphic-target scenario specifically
against one of *that type's actual existing instances*, not just a
freshly-created instance of a similarly-shaped test type — the two can
give opposite answers. If it fails, the fallback is one direct
relationship declaration per concrete target type (reusing one
relationship-type label across all of them) instead of a single
polymorphic one.

### 7.10 Same field shape is not the same BFO category — don't merge on convenience alone
Two types can end up with an identical property list (say, `name` +
`notes`, nothing else) for entirely different ontological reasons — one
because it's a Universal (a kind/type, like a product classification),
the other because it's a Particular that just happens to be simple (an
actual business, a specific supplier, a real person). It's tempting to
fold both into one shared abstract trait purely because deduplicating
identical-looking declarations feels like the obviously correct move.
Don't, if the categories genuinely differ: keep asking "is this data a
kind of thing, or an actual instance of something?" before merging.
🔧 Field-tested: in one real schema, six true Universals (reference/kind
types like fertilizer and treatment classifications) were correctly
merged onto one shared trait — but a seventh type with the *exact same*
two-field shape was a Supplier, a particular business, not a kind of
anything, and was deliberately left out of the merge despite matching
the field list perfectly. The technical dedup opportunity was real for
six of the seven; applying it to all seven anyway would have been
letting Structr's (or any platform's) surface-level shape-matching drive
an ontological decision it has no basis to make.

### 7.11 Planning: representing a process that hasn't happened yet
§7.3's Process-as-reification pattern covers events that already
occurred. It has no answer for "we intend to sow 200 tomato seedlings
starting April 1" — and reaching for BFO's Process Profile (§4) doesn't
help either, since that's about *measuring* an already-occurring
process, not scheduling a future one. The established starting point
is IAO's Plan Specification (the IAO term appears in over 40 OBO Foundry
ontologies, per the 2024 paper cited in §9), most often used in a
three-part pattern that paper calls the **RC-account**
("realizable concretization"). Read the caveat at the end of this
section before adopting it: that paper concludes the full RC-account
should *not* be used as-is.

1. **Plan Specification** (IAO_0000104) — a Generically Dependent
   Continuant (§3). IAO defines it as a directive information entity
   with action specifications and objective specifications as parts,
   which may be concretized as a realizable entity whose realization is
   a process in which the bearer tries to achieve the objectives by
   taking the specified actions. This is your **reusable
   recipe/template**: it has_part **Action Specifications** (the steps)
   and an **Objective Specification** (the intended endpoint). Example:
   a baking recipe is the Plan Specification, "mix, bake, cool" are
   Action Specifications, "produce edible cookies" is the Objective
   Specification.
2. **Plan** — a Realizable Entity (§3) borne by the agent who intends to
   carry it out, that `concretizes` the Plan Specification for **one
   specific occasion**: this particular baking session, not the recipe
   in general. One Plan Specification can be concretized into many
   Plans over time (the same recipe, used again next season).
3. **Planned Process** — an ordinary Process (§4) that `realizes` the
   Plan. It's not a separate process *type* — any process realizes a
   plan simply by having that relationship set; the actual baking event
   is a completely normal Process that happens to also point back at
   the Plan it fulfilled.

**The load-bearing insight, and the actual reason this needs its own
category at all**: a plan for a not-yet-existing thing cannot reference
that thing, because it doesn't exist yet. If your event-sourced schema's
every Process already requires a real participant (an Entry id, a
customer id, ...), a future-dated instance of that same event type
still can't be created — there's no real participant to attach it to.
The Plan instead references the *kind* (a Universal — a product/variety/
recipe classification) and a *quantity*, never a specific particular,
until a real event eventually happens and supplies one.

**The caveat: the source paper argues against the full RC-account.**
Duncan, Diller, Dooley, Hogan & Beverley (2024, §9) evaluated the RC-account and
found it inadequate for **failed plans** (if execution doesn't carry
out the specified actions, it's unclear how to relate the failed
process to the specification or its bearer) and for **changing plans**
(deciding on one course of action, then abandoning it for another).
They conclude it should not be used as-is and propose two alternatives:

- **Two hops, via `prescribed by`** (their recommended default, which
  they hypothesize is adequate for most situations): link the Plan
  Specification directly to the entity or process that uses it as a
  guide, using the Common Core Ontologies (CCO) `prescribed by`
  relation. No intermediate realizable. This also matches §7.8's
  "don't build ahead of a need."
- **A richer model** that adds the *process of forming a plan* (a plan
  in the sense of an intention to execute a specification). Use this
  only when your domain must represent a plan being formed, abandoned,
  and replaced before anything is executed.

Either way, don't make the planned-to-actual link itself carry a
pass/fail judgment. Link the real process to the specification it
followed (successful or not), and leave the comparison between what was
planned and what actually happened to a separate query against both
nodes' own data, not a state the schema computes or stores. 🔧 Field-tested:
implemented exactly this way — completing a plan links the real event
and advances the plan's status to "fulfilled" unconditionally; whether
the real event's actual quantity matched the plan's target is left for
whoever's asking to compare, the same way a shop-floor system compares
estimated vs. actual labor time after the fact rather than the system
adjudicating "success."

## 8. The 25 best-practice design principles (condensed from the book, chs. 3–4)

**General attitude (ch. 3):**
1. **Realism** — represent reality, not your current beliefs or a
   convenient fiction.
2. **Perspectivalism** — multiple accurate descriptions of the same
   reality can coexist (don't force one "true" model when two
   complementary ones both work, e.g. continuant vs. occurrent).
3. **Fallibilism** — your ontology will be wrong somewhere; version it
   and make correction cheap.
4. **Adequatism** — take every level of granularity seriously on its
   own terms; don't reductively explain away entities that don't fit
   your favorite level.
5. **Reuse** — don't reinvent a category that already has a good,
   reusable answer (this is the entire reason to start from BFO/OBO
   Foundry types instead of from scratch).
6. Balance utility and realism — don't sacrifice long-term adequacy for
   short-term convenience.
7. Ontology design is open-ended — plan for revision, not a one-shot
   deliverable.
8. **Low-hanging fruit** — start with what's easiest and clearest, work
   outward to the harder/controversial parts (directly supports §7.8).

**Terminology (ch. 4):**
9. Gather/select terminology from real usage, not invented labels.
10. Ensure maximal consensus with domain experts' actual vocabulary.
11. Identify areas of disciplinary overlap/inconsistent usage early.
12. Don't reinvent the wheel — stay close to existing usage and
    existing ontologies.
13. Use singular nouns for universals/types.
14. Use lowercase for common nouns (reserve capitals for proper names of
    actual instances).
15. Avoid acronyms in the canonical term (keep them as synonyms if
    unavoidable).
16. Give every term a stable, unique alphanumeric identifier — decouples
    the identifier from a label that might later need to change.
17. Ensure univocity — one term, one meaning, always (including
    relation names like `is_a`/`part_of` — don't let them silently mean
    different things in different corners of the schema).
18. Avoid mass nouns ("tissue," "water") — reformulate as "portion of X"
    so instances are actually countable.
19. Distinguish general (ontology) from particular (data) — keep your
    schema's terms and your actual instance data in clearly separate
    artifacts (T-box vs. A-box).

**Definitions (ch. 4):**
20. Give every non-root term a definition.
21. Use the Aristotelian form: *S = def. a G that Ds* (species = genus +
    differentia) — this is also a mechanical check on your `is_a`
    hierarchy, since every definition should name its immediate parent.
22. Define using essential features only (what the entity *must* have to
    be that kind of thing, not incidental/variable features).
23. Start from the most general terms and work down.
24. Avoid circularity (the definiendum, or a close synonym, can't appear
    in its own definiens).
25. Ensure intelligibility — definitions should use simpler/more basic
    terms than the term being defined, and every definition must be
    "unpackable" (substitutable for the term without changing meaning
    or breaking grammar).

**Taxonomy (ch. 4, cont'd):**
- Structure every ontology around one backbone `is_a` hierarchy (a
  directed rooted tree).
- Ensure `is_a`-completeness — every term traces to the root by a real
  chain of `is_a` edges.
- **Assert single inheritance** in your backbone hierarchy (each term
  has exactly one parent) even if your platform *supports* multiple
  inheritance — build any needed cross-cutting classification (e.g., by
  color *and* by vehicle type) as a separate, additional hierarchy, then
  let a reasoner/application layer combine them for specific uses,
  rather than tangling both into one asserted tree. This is the single
  best debugging tool available: when a term is hard to place, forcing
  a single parent is exactly what surfaces the ambiguity.
- Respect the open-world assumption — absence of a term/fact in your
  ontology is not evidence it's false, just unrecorded (the opposite of
  how a typical relational database with a closed-world "if it's not in
  the DB it's false" mental model behaves — see §8 below for why this
  matters for reasoners specifically).
- Adhere to objectivity — describe what exists, not what's currently
  known/measured/diagnosed (don't create classes like "unclassified
  influenza" or "known allergy" — that's conflating the disease with the
  diagnosis of it).

## 9. Tools & reference links

**Primary sources:**
- [Building Ontologies with Basic Formal Ontology](https://mitpress.mit.edu/9780262527811/building-ontologies-with-basic-formal-ontology/) — Arp, Smith & Spear, MIT Press, 2015. The standard textbook this cheat sheet condenses.
- [BFO 2020 GitHub repository](https://github.com/BFO-ontology/BFO-2020) — canonical OWL/CLIF/Prover9 artifacts, ISO/IEC 21838-2 conformant.
- [BFO 2020 terms table (xlsx)](https://github.com/BFO-ontology/BFO-2020/blob/master/21838-2/owl/profiles/temporal%20extensions/temporalized%20relations/documentation/bfo-2020-terms.xlsx) — every term's official elucidation/definition/examples/domain/range/ID. Authoritative; more current than the 2015 book.
- [BFO 2020 relations table (xlsx)](https://github.com/BFO-ontology/BFO-2020/blob/master/21838-2/owl/profiles/temporal%20extensions/temporalized%20relations/documentation/bfo-2020-relations-table.xlsx) — every relation with its inverse and temporalized (at-all-times/at-some-time) variants.
- [BFO Case Studies (PDF)](https://github.com/BFO-ontology/BFO-2020/blob/master/documentation/user%20guides/ontology%20developer%20guide/Basic%20Formal%20Ontology%20Case%20Studies.pdf) — worked classification examples across many domains.
- [basic-formal-ontology.org](https://basic-formal-ontology.org/) — official project homepage (now served from bfo-ontology.github.io).
- [OBO Foundry](https://obofoundry.org/) — the ecosystem of BFO-conformant domain ontologies; browse before building anything from scratch (principle of reuse, §8).
- ISO/IEC 21838-1:2021 (top-level ontologies, general requirements) and ISO/IEC 21838-2:2021 (BFO as the specific standard) — cite these if you need a standards reference for a compliance-minded stakeholder.

**Building/editing tools:**
- [Protégé](https://protege.stanford.edu/) — the standard free ontology editor (OWL-based); import BFO's latest OWL directly from `http://purl.obolibrary.org/obo/bfo.owl`.
- [ROBOT](https://github.com/ontodev/robot) — the OBO community's command-line tool for building, reasoning over, and releasing ontologies (actively maintained). OBO-Edit, the older OBO-format editor, is no longer maintained and its site is gone.
- Reasoners: [HermiT](http://www.hermit-reasoner.com/), [FaCT++](http://owl.man.ac.uk/factplusplus/), and [Pellet](https://github.com/stardog-union/pellet) (open-source repo, no commits since 2017; the old clarkparsia.com page is gone) — check consistency and compute inferred hierarchies. HermiT and ELK ship as Protégé plugins.
- Browsers: [BioPortal](https://bioportal.bioontology.org) (huge, cross-ontology term search — check here before inventing a term), [Ontobee](https://ontobee.org/) (OBO Foundry's linked-data term server).
- Reuse tools: [OntoFox](https://ontofox.hegroup.org/introduction.php) (MIREOT — pull in just the terms you need from an existing ontology rather than importing/copying the whole thing).

**Practical/graph-DB-adjacent reading found while researching this:**
- [W3C: Defining N-ary Relations on the Semantic Web](https://www.w3.org/TR/swbp-n-aryRelations/) — the generic version of the reification pattern in §7.3.
- [Wikipedia: Basic Formal Ontology](https://en.wikipedia.org/wiki/Basic_Formal_Ontology) — quick orientation/history.

**Planning (§7.11) and Process Profile (§4):**
- Smith, *Classifying Processes: An Essay in Applied Ontology*, [*Ratio* 25:4 (2012), 463–488](https://onlinelibrary.wiley.com/doi/10.1111/j.1467-9329.2012.00557.x) — origin of the Process Profile proposal; confirmed not carried into the ratified BFO spec.
- [Information Artifact Ontology (IAO)](https://github.com/information-artifact-ontology/IAO) — Plan Specification/Objective Specification/Action Specification definitions.
- [Ontology for Biomedical Investigations (OBI)](https://obi-ontology.org/) — Planned Process, and the broader material-entity/quality/planned-process/information-artifact modeling split worth skimming before designing any "study/protocol/experiment"-shaped domain.
- Duncan, Diller, Dooley, Hogan & Beverley, ["Concretizing plan specifications as realizables within the OBO foundry"](https://pmc.ncbi.nlm.nih.gov/articles/PMC11334599/) (*J. Biomedical Semantics*, 2024; PMID 39160586) — names and evaluates the RC-account, finds it inadequate for failed and changing plans, and proposes CCO `prescribed by` as the simpler alternative. The direct source for §7.11.

## 10. Common pitfalls (quick-scan list)

- Modeling a **participant in an event as a free-text string** instead
  of a real relationship — the single most common gap found by a
  field-by-field audit (§7.3).
- Modeling a **Role as a permanent boolean/type** instead of an
  assignable, revocable wrapper (§7.2/§3).
- **Overwriting** a Quality's current value with no history, when the
  history is itself real data worth keeping (§7.1).
- **Mass nouns** left un-countable ("tissue," "fertilizer") instead of
  reformulated as "portion of X" (principle 18).
- **Negative/complement classes** ("noneukaryotic thing") — fine to
  define a class *using* negative information ("lacks a nucleus"), never
  fine to define a class as the logical complement of another.
- **Multiple inheritance smell** in your asserted backbone taxonomy —
  almost always means two orthogonal classification axes (e.g., by
  color and by vehicle-type) have been tangled into one hierarchy;
  split them (principle: assert single inheritance).
- Confusing **`is_a` (subtype)** with **`instance_of` (particular →
  universal)** — a live phenomenon called "is_a overloading," a classic,
  well-documented ontology bug.
- Confusing **a universal-hierarchy tree with a particular-hierarchy
  tree** when both happen to use the same self-referential parent-
  pointer shape in your schema — and, specifically, labeling a
  universal-hierarchy's real `is_a` relation `PART_OF` just because a
  particular-hierarchy sitting right next to it in the same schema
  correctly uses that label for real parthood (§7.6, field-tested).
- Building speculative structure (a full taxonomy, a lot-tracking
  system, a site hierarchy) before a real query needs it (§7.8).
- Trusting a platform's advertised inheritance/schema mechanism without
  testing it directly against your own running instance — and, even
  after confirming the mechanism itself, trusting a polymorphic
  relationship target to work against instances that already existed
  *before* the type gained the trait, without testing that specific,
  narrower case (§7.9, field-tested).
- Merging two types onto one shared abstraction purely because their
  field list matches, without checking whether they're actually the same
  BFO category (a Universal and a Particular can coincidentally have
  identical fields) (§7.10, field-tested).
- Reaching for **Process Profile** to model a future/planned process — it
  was proposed for BFO 2.0 to represent a measurable *dimension* of a
  process already underway (rate, rhythm), was ultimately left out of
  the ratified spec, and doesn't answer the scheduling question anyway.
  Start from IAO's Plan Specification instead (§7.11).
- Reaching for the heavier **reified-Role-node** pattern (§7.2) by
  default, even when the role is fully exhausted by "which particular,
  in which specific circumstance" and needs no state of its own — a bare
  relationship on the one process that actually realized the role is
  enough, and simpler (§7.2, field-tested).
- Trying to represent a plan for a **not-yet-existing particular** by
  attaching a future-dated instance of an ordinary event/Process type —
  it has nowhere to put the participant that doesn't exist yet. That's
  the actual signal you need a Plan Specification (§7.11), not a
  workaround on the existing event type.
