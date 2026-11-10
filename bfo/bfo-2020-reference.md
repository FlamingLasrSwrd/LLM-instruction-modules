# BFO 2020: class and relation reference

A compact lookup table for BFO 2020 (ISO/IEC 21838-2) classes and relations,
with their declared domains and ranges, plus the IAO, CCO and RO terms most
often used beside them. Companion to [`bfo-cheatsheet.md`](bfo-cheatsheet.md),
which explains how to apply them.

> **Last verified 2026-11-10** against: BFO-2020 `master` (commit `bd1fb61`,
> 2026-10-06) terms and relations tables and `bfo-temporalized-relations.owl`;
> IAO `iao-edit.owl`; RO `ro-edit.owl`; CCO `develop`
> `InformationEntityOntology.ttl`. Domains and ranges are as those files
> declare them. Examples marked *(BFO)* are from the official terms table;
> the rest are illustrative.

---

## 1. Classes

### 1.1 Independent continuants

Persist through time and exist wholly at every moment they exist.

| Class | Elucidation (condensed) | Examples |
|---|---|---|
| **Entity** | The root | anything |
| **Continuant** | Persists through time, exists wholly at each moment, may change | a bag of flour, a knife, a recipe |
| **Independent continuant** | Does not depend on another entity to exist; bears qualities | an apple, a pan |
| **Material entity** | Independent continuant that at all times has some portion of matter as part | a human being, an undetached arm, an aggregate of human beings *(BFO)* |
| **Object** | Material entity that manifests causal unity and is maximal relative to that kind of unity | an organism, a fish tank, a laptop, an ice cube *(BFO)* |
| **Object aggregate** | Material entity consisting exactly of a plurality of objects as member parts which together form a unit | the musicians in an orchestra, a collection of cells in a biobank *(BFO)*; a dozen eggs |
| **Fiat object part** | Proper part of an object, demarcated from the rest of it by fiat surfaces | the lobes of the left lung, the Western hemisphere of the Earth *(BFO)* |
| **Immaterial entity** | Independent continuant with no material entity as part | a cave, the hold of a ship |
| **Site** | 3-D immaterial entity whose boundaries coincide with, or are located relative to, material entities | a rabbit hole, the Piazza San Marco, the hold of a ship *(BFO)*; the interior of a fridge |
| **Continuant fiat boundary** | 0-, 1- or 2-D immaterial entity located relative to some material entity | the North Pole, a national border |
| **Spatial region** | A part of space itself | a cubic metre of space |

### 1.2 Specifically dependent continuants (SDC)

Cannot exist without one **specific** bearer; if the bearer goes, it goes.

| Class | Elucidation (condensed) | Examples |
|---|---|---|
| **Specifically dependent continuant** | Depends on a specific bearer; cannot migrate | the mass of the Eiffel Tower |
| **Quality** | An SDC that, unlike roles and dispositions, needs no further process to be realized | the colour of a tomato, the mass of this piece of gold *(BFO)*; the cleanliness of a pan |
| **Relational quality** | A quality that depends on two or more bearers | a marriage bond, an obligation between two people *(BFO)* |
| **Realizable entity** | An SDC exhibited only in certain processes | fragility; a doctor's role |
| **Role** | Realizable, **externally grounded**: exists because of circumstances the bearer need not be in; losing it changes nothing physical about the bearer | the student role, the role of a stone in marking a property boundary *(BFO)*; being reserved for Wednesday's lunch |
| **Disposition** | Realizable, **internally grounded**: if it ceases, the bearer is physically changed | an atom's disposition to decay *(BFO)*; the fragility of a glass |
| **Function** | A disposition the bearer has because it was designed or evolved to realize it | a heart's pumping function, a hammer's function to drive nails *(BFO)* |

### 1.3 Generically dependent continuants (GDC) and information entities

Copiable content or pattern: exists because there is at least one of
possibly many copies.

| Class | Source | Elucidation (condensed) | Examples |
|---|---|---|---|
| **Generically dependent continuant** | BFO | The content or pattern that copies share | a PDF file and its copy, the content of a sentence, an engineering blueprint *(BFO)* |
| **Information content entity** (IAO_0000030) | IAO | A GDC that is *about* something | a document, a database record, a measurement datum |
| **Prescriptive information content entity** | CCO | An ICE that prescribes some entity. Older CCO releases called this *Directive ICE* | a recipe, a protocol, an order |
| **Designative information content entity** | CCO | An ICE consisting of symbols that designate some entity | a name, an identifier |
| **Descriptive information content entity** | CCO | An ICE that describes some entity | a measurement result, a price record |
| **Plan specification** (IAO_0000104) | IAO | A directive information entity with action and objective specifications as parts | a recipe, a study protocol |

> **Constraint:** a GDC cannot be the bearer of a role, disposition or
> function. In BFO 2020, `inheres in` has range *independent continuant that
> is not a spatial region*, so no SDC (realizables included) can inhere in a
> plan, an identifier, a measurement or any other ICE.

### 1.4 Occurrents

Unfold in time and have temporal parts.

| Class | Elucidation (condensed) | Examples |
|---|---|---|
| **Occurrent** | Happens, rather than persists | a game, a cooking session |
| **Process** | Occurrent with a proper temporal part that has at least one material participant | an act of selling, a beating of the heart *(BFO)*; washing a pan |
| **Process boundary** | Temporal part of a process with no proper temporal parts | the boundary between the 2nd and 3rd year of your life *(BFO)*; the moment the oven reached 200 °C |
| **History** | The sum of all processes in the spatiotemporal region occupied by a material entity's material part | the life of an organism *(BFO)* |
| **Temporal region** | A part of time | the year 2026; 4:00 to 6:00 PM Saturday |
| **Zero-dimensional temporal region** (temporal instant) | An instant | midnight |
| **One-dimensional temporal region** (temporal interval) | An interval | Saturday afternoon |
| **Spatiotemporal region** | A region of space-time | the region a game occupies |

There is no *process profile* class in BFO 2020.

---

## 2. Relations, with declared domain and range

### 2.1 Dependence and inherence (BFO)

| Relation | Domain | Range | Notes |
|---|---|---|---|
| `specifically depends on` | SDC | independent continuant that is not a spatial region, or an SDC | parent of `inheres in`. Inverse `specifically depended on by` |
| `inheres in` | SDC | independent continuant that is not a spatial region | *a mass inheres in a material entity (BFO)*. Inverse `bearer of` |
| `generically depends on` | GDC | independent continuant that is not a spatial region | *this PDF's content generically depends on some disk*. Inverse `is carrier of` |
| `concretizes` | SDC or process | GDC | *the patterns of ink in this copy of War and Peace concretize the novel (BFO)*. A process in the domain means an execution can concretize a plan. Inverse `is concretized by` |

### 2.2 Parthood, membership, location (BFO)

| Relation | Domain | Range | Notes |
|---|---|---|---|
| `continuant part of` / `has continuant part` | continuant | continuant | Temporalized in BFO 2020-OWL as `... at all times` / `... at some time`; only the at-all-times form is declared transitive |
| `member part of` / `has member part` | object | object aggregate | BFO-native membership (each egg in a dozen) |
| `occurrent part of` / `has occurrent part` | occurrent | occurrent | a sub-process of a cooking session |
| `located in` / `has location` | independent continuant that is not a spatial region | same | **Containment, not parthood**: flour in a bag is located in it, not part of it |

### 2.3 Participation, realization, time (BFO)

| Relation | Domain | Range | Notes |
|---|---|---|---|
| `participates in` | continuant that is not a spatial region (IC, SDC or GDC) | process | |
| `has participant` | process | continuant that is not a spatial region | inverse |
| `realizes` | process | realizable entity | *a role of student is realized in studying processes (BFO)* |
| `has realization` | realizable entity | process | inverse (BFO 2.0 and some tools call it `realized in`) |
| `occupies temporal region` | process or process boundary | temporal region | |
| `precedes` / `preceded by` | occurrent | occurrent | temporal order |
| `history of` / `has history` | history | material entity | each material entity has exactly one history |

### 2.4 Aboutness (IAO, CCO)

| Relation | Source | Domain | Range | Notes |
|---|---|---|---|---|
| `is about` (IAO_0000136) | IAO | information content entity | unrestricted | IAO calls it a "currently primitive" relation. Domain-specific aboutness is usually modeled as subproperties of it |
| `designates` | CCO | designative ICE | entity | the naming/identifying kind of aboutness |
| `prescribes` / `prescribed by` | CCO | prescriptive ICE / entity | entity / prescriptive ICE | the ICE "serves as a rule or guide" for an occurrent or "a model" for a continuant. Recommended by the 2024 RC-account paper for linking an execution, including a failed one, to the plan it followed (see the cheat sheet §7.11) |

### 2.5 RO participation and existence refinements

In `ro-edit.owl` only `has input` declares a domain (process); the
domains and ranges below are the intended usage from each relation's
definition, not OWL axioms.

| Relation | ID | Domain | Range | Notes |
|---|---|---|---|---|
| `has input` | RO_0002233 | process | continuant | consumed or acted on |
| `has output` | RO_0002234 | process | continuant | produced |
| `existence starts during` | RO_0002488 | continuant | occurrent | *this leftover's existence starts during the portioning*. Inverse-direction and "or after"/"or before" variants exist |
| `existence ends during` | RO_0002492 | continuant | occurrent | |
| `derives from` | RO_0001000 | continuant | continuant | from RO2005; RO notes it may be replaced by the "develops from" family |
| ~~`has agent`~~ | RO_0002218 | | | **Obsolete.** Now labeled "obsolete has active participant"; "has agent" survives only as an alternative label. It meant the entity causally responsible for a process (an organism or agent), not an instrument such as an oven |
