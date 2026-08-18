---
title: Reading merge history as an event log
parent: Snippets
---

# Reading merge history as an event log

The technique from [case study 01](../case-studies/01-identity-disaggregation.md). A CRM records
absorbed record IDs as a cumulative, delimiter-joined string, versioned over time. Diffing consecutive
versions turns that into an ordered log of individual merge events with timestamps, using no API
surface beyond property history.

```python
def merge_events(history):
    """
    history: property versions for the cumulative merged-IDs field,
             oldest first, each {"value": "101;102;103", "timestamp": ...}
    yields:  one event per record actually absorbed, with when it happened
    """
    seen = set()
    for version in history:
        ids = {i for i in version["value"].split(";") if i}
        for new_id in ids - seen:
            yield {"absorbed_id": new_id, "at": version["timestamp"]}
        seen |= ids
```

The same shape works for any cumulative field: tags, list memberships, applied labels. The value is
that a field designed to answer "what is true now" also answers "what happened, and when", provided
the platform versions it.

## Attributing values with the delta, not the snapshot

A merge snapshot is the survivor's blended post-merge state and describes no real record. Only the
values that *changed* at the merge timestamp came from the absorbed record.

```python
def absorbed_record_values(property_histories, merge_timestamp):
    """Values that genuinely came from the record absorbed at this timestamp."""
    out = {}
    for prop, versions in property_histories.items():
        for i, v in enumerate(versions):
            if v["timestamp"] != merge_timestamp:
                continue
            previous = versions[i - 1]["value"] if i else None
            if v["value"] != previous:          # changed: came from the absorbed record
                out[prop] = v["value"]
            # unchanged at the merge: the survivor's own value echoing. Skip it.
    return out
```

Skipping the unchanged values is the whole difference between reconstructing a person and inventing
one.
