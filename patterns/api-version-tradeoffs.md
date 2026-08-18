---
title: API version trade-offs
parent: Patterns
---

# API version trade-offs

**Rule.** Do not assume the current API version is a superset of the one it replaced. Check what the
legacy endpoint returns before concluding that data is unrecoverable.

**The case.** A platform's current API caps property history at 45 versions per property. On a record
merged hundreds of times, the oldest identities are simply absent from that response, and absent
without any indication that truncation occurred. Reading only the modern API, the reasonable conclusion
is that the data is gone.

The platform's legacy endpoint returns an uncapped identity payload. On one record where the modern API
exposed 44 identities, the legacy endpoint returned 1,001. The entire feasibility of the project turned
on that difference.

**Why this happens.** New API versions are designed around expected access patterns, and caps are added
for performance against those patterns. Forensic and recovery work is not an expected pattern. The
legacy endpoint predates that optimization and often exposes the raw structure, which is exactly what
recovery needs.

**How to check.** For any claim that history or detail is unavailable, enumerate every endpoint that
touches the object, including deprecated ones, and compare their payloads on a record you know is
extreme. Caps show up immediately at the extreme and are invisible on a typical record.

**Use both, deliberately.** The pragmatic design is the modern API for bulk reads because it batches
and is supported, and the legacy endpoint for the specific payload only it provides, with a comment
explaining precisely why the legacy call exists. Undocumented legacy dependencies are how a working
tool breaks silently a year later when the endpoint is finally retired.

**Evidence:** [Case study 01](../case-studies/01-identity-disaggregation.md).
