---
title: Spreadsheets corrupt identifiers
parent: Patterns
---

# Spreadsheets corrupt identifiers

**Rule.** A spreadsheet infers types. An identifier that passes through one may stop being that
identifier, silently, and never error. Keep identifier columns as text at every hop, and do not let a
spreadsheet be a hop in any pipeline that carries keys.

**The specific failure.** A column of digits gets inferred as numeric, and leading zeros are stripped.
`0041827` becomes `41827`. The value still looks like an identifier, is one character shorter, and
matches nothing.

Nothing warns. The file saves, the import succeeds, and the damage appears later as an inexplicably
low match rate for one segment of the population, with the cause two steps upstream in a tool nobody
considered part of the pipeline.

**Other shapes of the same problem:**

- Long numeric IDs converted to scientific notation, losing precision in the low digits.
- Strings that look like dates being reformatted. Locale settings decide what the value becomes.
- Trailing or leading whitespace introduced or stripped inconsistently, which breaks exact joins.
- Character encoding changed on save, so names with accents stop matching.

**The defenses, in order of preference.**

1. **Do not route keys through a spreadsheet.** Most of the time the spreadsheet is a convenience, not
   a requirement, and removing it removes the whole class of problem.
2. **Force identifier columns to text** on both import and export when a spreadsheet is unavoidable.
3. **Verify format, not just presence.** Check the length distribution of identifier columns against
   the source before importing. A key that is supposed to be seven characters and is sometimes six
   tells you immediately, and this is a check that costs seconds.

**Why it belongs in a patterns library.** It looks like a beginner problem and it is not: it survives
in sophisticated pipelines because the corrupting step is a person opening a file to look at it. The
cost is high because the failure is silent and the diagnosis path leads through the data and the
importer before it leads to the spreadsheet.

**Related:** a corrupted key defeats [composite match keys](composite-match-keys.md) completely. The
key generates, looks valid, and matches nothing.

**Evidence:** [Case study 10](../case-studies/10-migration-at-scale.md), where this corrupted
identifiers during a multi-million record migration and was found by investigating a low match rate,
which is to say, found late.
