# The Unofficial Guide

Jorge Maldonado — corpus: `city_guides`

---

# Unit 1

## What This Does

This system answers questions about `city_guides`, a corpus of 14 travel guides covering nine towns plus five cross-cutting guides (eating, walking, transport, seasons, accessibility). It answers specific, factual questions like whether public transport reaches a town, when a market runs, or which street to eat on to avoid tourist prices — grounded in the guides themselves, with sources named. Questions the guides don't cover (like "what is the capital of Mongolia") are refused rather than answered from the model's own knowledge.

## Chunking Strategy

**Chunk size:** variable, capped at 700 characters, split by markdown heading section (average came out to 290 characters across 99 chunks)
**Overlap:** none — heading boundaries are used instead of overlapping windows

`city_guides` documents are organized under `##` headings (Getting there, Eat and drink, Where to stay, etc.), each covering one topic in a paragraph or two. The starter's fixed 800-character window chunker cut straight through these headings — 14 documents became 51 chunks, and the shortest was 24 characters (just a stray heading fragment). I replaced it with a chunker that splits on `##` headings first, so each section stays intact as one topic, and only sub-splits a section by paragraph if it exceeds 700 characters. This produced 99 chunks averaging 290 characters (min 23, max 637).

The shortest chunk is still under my 50-character target from `criteria.md` — a few documents open with a bare `#` title line before their first `##` heading, and my merge-back logic only merges a too-short piece into the previous chunk from the same document, so a document's very first section has nothing to merge into. I'm leaving this as a known limitation given time constraints — a real, diagnosable gap rather than something I'm hiding.

## Sample Chunks

**Chunk 1** — source: `guide_accessibility.md#0` — produced by: `chunker.py::split_documents`

Getting around the region with limited mobility. An honest assessment rather than a promotional one. Some of these places are difficult and it is better to know in advance.

**Chunk 2** — source: `guide_corry_vale.md#5` — produced by: `chunker.py::split_documents`

Where to stay. Perhaps thirty beds in the entire valley, spread across two pubs and a handful of farmhouse rooms. In summer these are booked months ahead. Camping is permitted on two marked fields and nowhere else.

**Chunk 3** — source: `guide_givens_mill.md#2` — produced by: `chunker.py::split_documents`

Getting around. Everything is on one street along the river. The mill is at one end and the church at the other, eight minutes apart. The riverside path continues in both directions for as far as you want to walk.

**Chunk 4** — source: `guide_kestrelford.md#5` — produced by: `chunker.py::split_documents`

Where to stay. Two inns on the square and a handful of rooms above the pubs. Booking ahead matters between May and September and not at all otherwise. There is no accommodation of any kind within four miles of the town in either direction.

**Chunk 5** — source: `guide_regional_transport.md#0` — produced by: `chunker.py::split_documents`

Getting around the region.

Chunk 5 shows the known limitation described above — a bare title heading with no body text became its own chunk, since it was the first section in the document with nothing to merge backward into.

## Sample Answer

**Question:** Does the Kestrelford bus run on Sundays?

**Answer:** No, the Kestrelford service does not run on Sundays (guide_regional_transport.md). Sources retrieved: guide_brightwater.md, guide_eating.md, guide_givens_mill.md, guide_regional_transport.md. Best distance 0.262, cutoff 0.6.

**My relevance cutoff:** 0.6 (the starter's default)

I ran my 5 test questions and the 5 OUT_OF_SCOPE questions and recorded the best distance for each. In-corpus questions clustered between 0.262 and 0.540; out-of-scope questions clustered between 0.754 and 0.899 — a clean gap with nothing in between. 0.6 sits comfortably in that gap, so I kept the default.

| Question | In corpus? | Best distance |
|---|---|---|
| Is there public transport to Elder Ness? | Yes | 0.351 |
| Is it safe to swim at the beach in Elder Ness? | Yes | 0.540 |
| Does the Kestrelford bus run on Sundays? | Yes | 0.262 |
| Where should you eat in Brightwater to avoid tourist prices? | Yes | 0.437 |
| What day of the week is hardest to find dinner in this region? | Yes | 0.489 |
| What is the capital of Mongolia? | No | 0.754 |
| How do I change the oil in a diesel engine? | No | 0.892 |
| Who won the 1994 World Cup? | No | 0.899 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.846 |
| How do I write a for loop in Rust? | No | 0.813 |

## How I Used AI

**1.** I asked Claude to help me draft a chunking strategy for city_guides after describing that the documents use markdown headings to organize sections. It suggested splitting on headings first and only sub-splitting a section by paragraph if it ran long. I used that structure but added the merge-back logic for too-short fragments myself after noticing a 23-character heading-only chunk in my output — the original suggestion didn't handle that case.

**2.** I asked Claude to pressure-test my five acceptance criteria before committing them. It flagged that my original chunk-length criterion ("no chunk under 10 chars") was already guaranteed to pass since I'd already observed a 24-character chunk, meaning the target couldn't fail. I raised it to 50 characters instead.
