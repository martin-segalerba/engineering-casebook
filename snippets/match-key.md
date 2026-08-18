---
title: Building and auditing a match key
parent: Snippets
---

# Building and auditing a match key

From [case study 02](../case-studies/02-crm-data-architecture.md). The key itself is trivial. The part
worth writing down is auditing completeness before you rely on it.

```python
COMPONENTS = ("first_name", "last_name", "birthdate", "zip", "sex")

def match_key(record, components=COMPONENTS):
    """
    Returns the key, or None if any component is missing.
    Returning None is deliberate: a key with a blank component matches
    nothing, silently, and looks like a data problem rather than a key problem.
    """
    parts = []
    for c in components:
        v = (record.get(c) or "").strip().lower()
        if not v:
            return None
        parts.append(v)
    return "|".join(parts)


def completeness(records, components=COMPONENTS):
    """Report which components are blocking key generation, before importing anything."""
    from collections import Counter
    missing, keyable = Counter(), 0
    for r in records:
        gaps = [c for c in components if not (r.get(c) or "").strip()]
        if gaps:
            for c in gaps:
                missing[c] += 1
        else:
            keyable += 1
    return {
        "total": len(records),
        "keyable": keyable,
        "keyable_pct": round(100 * keyable / max(len(records), 1), 1),
        "missing_by_component": dict(missing),
    }
```

Run `completeness()` first. It tells you the size of the population that structurally cannot match, and
which single component is responsible for most of it. Without it, that population shows up at the end
of the import as an unexplained orphan count.

## Confirming a crosswalk before trusting it

The check that rejected 698 proposed merges in
[case study 01](../case-studies/01-identity-disaggregation.md).

```python
def bridge_is_confirmed(left, right):
    """
    Two records linked by a shared ID are the same person only if an
    attribute the join did NOT use also agrees. Birth year first,
    last name where birth year is missing.
    """
    ly, ry = left.get("birth_year"), right.get("birth_year")
    if ly and ry:
        return ly == ry
    ln = (left.get("last_name") or "").strip().lower()
    rn = (right.get("last_name") or "").strip().lower()
    return bool(ln) and ln == rn
```

A perfect one-to-one crosswalk passes every structural check and can still be wrong, because
identifiers get reassigned. See [ID reuse is real](../patterns/id-reuse-is-real.md).
