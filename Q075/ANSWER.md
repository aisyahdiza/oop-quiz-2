# Q075 — Your answer

## (a) The trace

Rows are written as `id customer`. Default view: all orders, sorted by ready time (ties by id), no filter, every status. Settings persist from one event to the next, so each event is evaluated with all the settings set before it.

| # | Event | Rows the view-model computes | View |
| --- | --- | ----| ---|
| 0 | attach | A2 Budi, A4 Dewi, A5 Rahmat, A1 Sari, A6 Citra, A3 Ayu (ready 11, 12, 13, 14, 15, 16) | updated (first rows) |
| 1 | fiter "ra" | 	A5 Rahmat, A6 Citra (names containing "ra": Rahmat, Citra) | updated |
| 2 | 	sort by customer | 	A6 Citra, A5 Rahmat (C before R) | updated |
| 3 | 	add A7 Budi (IR, RECEIVED, 17:00) | same two rows: "Budi" does not contain "ra" | no update |
| 4 | advance A5 (WASHING -> DRYING) | A6 Citra, A5 Rahmat (same order, A5's status text changed) | updated |
| 5 | filter "RA" | same two rows: matching ignores case, "RA" = "ra" | no update |
| 6 | only DRYING | A6 Citra, A5 Rahmat (both are DRYING now) | no update |
| 7 | advance A6 (DRYING -> READY) | A5 Rahmat (A6 no longer DRYING) | updated |
| 8 | 	filter cleared | 	A5 Rahmat (the only DRYING order; A1 is WASHING, A3 is RECEIVED, A2 and A4 are READY, A7 is RECEIVED) | no update |
| 9 | only READY | 	A2 Budi, A6 Citra, A4 Dewi (sorted by customer; A7 Budi is RECEIVED, not READY) | updated |

The view is updated 6 times for 10 events, which matches the prograp outpus `view updates: 6 for 10 events`.

## (c) Why compare

An event does not always change waht is on screen. Adding an order that the filter hides, re-typing the same filter in anothe case, or changing a setting that excludes nothing new all produce exactly the same rows. Notifying on every event would make the widget redraw, rebuild its rows and possibly lose the user's selection or scroll position, for no visible change. By comparing the new list with the last list shown (`rows.equals(shown)` in Jawa, `rows != self.shown` in Python) the view-model passes only real changes on to the view.

It is easy to test because the view-model has no widget in it. A test can therefore pass a function that appends the rows to a list, fire events on the model and the setters, and compare the list with the expected rows and the `updates` counter, with no window, no event loop and no screen. The `main` of this question does exactly that.

## (d) Chapter 9 ideas

- Collections: List for the orders and for the rows, with `List.copyOf` / `tuple` so the model hands out a read-only snapshot, and `List.equals` / `==` for comparing two row lists.
- A stream pipeline in Java (`filter`, `sorted`, `map`, `toList`) and a list comprehension with `sorted(key=...)` in Python.
- Comparators: a comparator chain (`comparingInt(...).thenComparing(id)`, `String.CASE_INSENSITIVE_ORDER`) and tuple sort keys in Python, so that ties are broken by order id.
- Generics: `Comparator<Order>`, `Consumer<List<String>>`, `List<Order>` and `List<Runnable>` for the listeners; the compiler checks the element types.
- Functional-style callbacks (lambdas and method references such as `this::recompute`) as the listeners.