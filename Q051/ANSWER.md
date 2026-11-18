# Q051 — Your answer

## (d) HashMap order vs Python dict order

A `HashMap` stores its entries in a table of buckets. The bucket of a key is computed from the key's `hashCode()` and the current table capacity, and `keySet()` simply walks the buckets from first to last. The Java specification states explicitly that `HashMap` makes no guarantee about the order of iteration and that the order may even change over time. The actual order therefore depens on implementation details: the hash function and bucket of insertion when keys collide. Another machine or JDK version may use different details, so the same program may print `[SH, IR, WDI, WF]` on one and a different order on another. A report that depends on this order is not that reliable.

A `dict` iterates in **insertion order**. This has been guaranteed by the language since Python 3.7. For our data it prints `WF, IR, WDI, SH`, the order in which the codes first appear in the log. This guarantee is **not** and not sorted by count. If we want a sorted report we must sort explicitly, e.g. `sorted(count)` for alphabetical order, as in part (a).

To fix the problem at Java, use a `TreeMap` when keys must come out sorted (what part (a) does), or a `LinkedHashMap` when they must come out in insertion order (the same guarantee as the Python `dict`).
