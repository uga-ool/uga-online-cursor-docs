---
name: Chunk Context Preambles
version: "1.0"
---

You are preparing text chunks for a vector search system. Below is a document and the boundaries of its chunks. For each chunk, write a 1-3 sentence context preamble that resolves any ambiguities a reader would face seeing the chunk in isolation.

## What to resolve

- **Pronouns**: "this", "they", "the author", "it" -- name the actual referent
- **Acronyms**: If an acronym is used in the chunk but was only defined earlier in the document, expand it
- **Cross-references**: "as discussed above", "the previous section", "see below" -- briefly state what was discussed or what follows
- **Implicit context**: What document, chapter, topic, or argument this chunk belongs to

## What NOT to do

- Do not summarize or rewrite the chunk content itself
- Do not repeat text from the chunk
- Keep each preamble under 100 words
- If a chunk is already self-contained and unambiguous, return an empty string for its preamble

## Response Format

JSON array only (no markdown fences, no commentary), one entry per chunk in order:

[
  { "chunkIndex": 0, "preamble": "This section is from Chapter 3 of..." },
  { "chunkIndex": 1, "preamble": "" },
  { "chunkIndex": 2, "preamble": "The 'entropy formula' referenced here is H = -Σ p log p, introduced by Claude Shannon..." }
]

## Document

{{document}}

## Chunks

Each entry below shows a chunk index followed by a preview of its text. Base your preambles on the document above and the chunk text, not on any external labels or summaries.

{{chunkList}}
