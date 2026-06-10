# Match / Search / Merge

A consolidated reference for the three duplicate-handling workflows. See [match.md](match.md), [search.md](search.md), and [merge.md](merge.md) for the per-topic versions.

## Match

The matching system compares two records and returns a confidence score in `[0.00, 1.00]` plus a per-component breakdown.

**Strategies**

- **Probabilistic** — weighted, fuzzy. Each component scores 0–1; the weighted sum is the overall score.
- **Deterministic** — rule-based. Short-circuit rules (e.g., exact tax-ID match) can pin the score to 1.0.

**Algorithms**

- **Jaro-Winkler** similarity — case-insensitive, prefix bonus, for name fields.
- **Levenshtein** distance — normalized by max length, for short strings.
- **Weighted field-by-field Jaro-Winkler** — addresses; only fields present in both contribute.
- **Haversine** distance with sigmoid decay — geo coordinates.
- **Soundex** phonetic code (4 chars) — applied as a +0.05 bonus when codes match and score < 0.95.

**Confidence classification** (defaults; thresholds are configurable)

| Quality  | Range         |
| -------- | ------------- |
| Certain  | ≥ 0.95        |
| Probable | ≥ 0.80–0.85   |
| Possible | ≥ 0.50–0.60   |
| Unlikely | below         |

## Search

- Full-text search across indexed fields
- Fuzzy search with configurable tolerance
- Phonetic search (Soundex) integrated into name matching
- Boolean query syntax (AND, OR, NOT)
- High-performance indexing with **Tantivy**
- Search by name, address, identifier, and entity-specific fields
- Geo-radius search where geo coordinates are modeled
- Automatic index synchronization with database writes
- Pagination (`offset` + `limit`)
- Option to mask sensitive data in search results

## Merge

- Merge confirmed duplicate records into a surviving **main** record
- Auto-merge for high-confidence matches; review queue otherwise
- Transfers identifiers, names, addresses, contacts, documents, and (where applicable) tax IDs
- Adds the duplicate's primary name as a "former" alias on main
- Creates a link (`Replaces`) from main → duplicate
- Marks the duplicate inactive via soft delete
- Maintains a merge history record with a snapshot of transferred data
- Publishes a `Merged` event on the event stream

## Duplicate detection

- **Real-time** on create — `POST /api/<plural>` returns `409 Conflict` with candidate matches when duplicates are detected
- **Explicit** check — `POST /api/<plural>/check-duplicates` checks without creating
- **Batch** — `POST /api/<plural>/deduplicate` scans the entire index
- **Review queue** — items captured with status `Pending`, `Confirmed`, `Rejected`, `AutoMerged`
- **Configurable rules** — `threshold`, `max_candidates`, `auto_merge_threshold`
