# Q054 — Your answer

## (a) The eight lines

TODO(Q054): for each of the eight lines, say whether it compiles and why
(totalKg(shirts); totalKg(anything); addSpareTowels(basket, 2); addSpareTowels(bathTowels, 1);
transfer(bathTowels, towels); transfer(towels, shirts); List<Garment> g = shirts;
view.add(new Shirt("x"))).

### Answer:

1. `totalKg(shirts);`
Compiles because `List<Shirt>` is a subtype of `List<? extends Garment>` (Shirt <: Garment).
2. `totalKg(anything);`
Does not compile. `List<Object>` is not `List<? extends Garment>`: because `Object` is not a subtype of `Garment`, but rather its superclass.
3. `addSpareTowels(basket, 2);`
Compiles, because `Garment` is a supertype of `Towel`, so `List<Garment>` is accepted as `List<? super Towel>`.
4. `addSpareTowels(bathTowels, 1);`
Does not compile. `BathTowel` is a subclass of `Towel`, not a superclass. Therefore, a `List<BathTowel>` cannot contain regular `Towel`s.
5. `transfer(bathTowels, towels);`
Compiles. T is interpreted as Towel: `List<BathTowel>` is a `Collection<? extends Towel>`, and `List<Towel>` is a `Collection<? super Towel>`.
6. `transfer(towels, shirts);`
Does not compile. Because it requires a `T` where `Towel <: T` (source) and `T <: Shirt` (destination). This implies an invalid `Towel <: Shirt` relationship, so there is no valid `T`.
7. `List<Garment> g = shirts;`
Does not compile, because generics are invariant. `List<Shirt>` is not a subtype of `List<Garment>` even though `Shirt <: Garment`.
8. `view.add(new Shirt(“x”));`
Does not compile. `view` is of type `List<? extends Garment>`. The compiler only knows that the elements are unknown subtypes of `Garment`, so there is no safe value to add except `null`.


## (c) The rule

TODO(Q054): state the PECS rule and apply it to `transfer`.

### Answer:

PECS stands for Producer Extends, Consumer Super.

- If a collection only produces values that we read, use `? extends T`.
- If a collection only accepts values that we write, use `? super T`.
- If used for both reading and writing, use an exact type or a type parameter.

In `transfer`, `src` is the producer (we read items from `src`), so its type is `Collection<? extends T>`.
`dst` is the consumer (we put items into it), so its type is `Collection<? super T>`. 
The type parameter `T` connects the two: anything read from `src` must be a `T`, and `dst` accepts any `T`.
That’s why `transfer(bathTowels, towels)` is valid (T = Towel) and `transfer(towels, shirts)` is invalid.


## (d) Python

TODO(Q054): which calls does mypy reject, and how do you express `? super Towel` in Python
(variance: covariant / contravariant / invariant)?

### Answer:

### Calls rejected by mypy:
- `total_kg(anything)`: `List[object]` is not a `Sequence[Garment]` because `object` is not a subtype of `Garment`.
- `add_spare_towels(basket, 2)` and `add_spare_towels(bath_towels, 1)`: `MutableSequence` is invariant, so only a matching `MutableSequence[Towel]` is accepted. Both `List[Garment]` and `List[BathTowel]` are rejected.
- `add_spare_towels_anywhere(bath_towels, 1)`: `append` in `List[BathTowel]` only accepts `BathTowel`, whereas `Sink[Towel]` requires the ability to accept a regular `Towel`.
- `g: List[Garment] = shirts`: `List` is invariant, just like in Java.
- `transfer(towels, shirts)` and `transfer([Towel("rogue")], only_shirts)`: `dst: List[T]` is invariant, so T is forced to be `Shirt`, and the source containing `Towel` does not match `Iterable[Shirt]`.
- `view.append(...)` if `view: Sequence[Garment]`: `Sequence` does not have an `append` method.

### Expressing `? super Towel` in Python

Python uses a contravariant type variable inside a `Protocol` because it does not have wildcards, as shown below:

```python
T_contra = TypeVar("T_contra", contravariant=True)

class Sink(Protocol[T_contra]):
    def append(self, item: T_contra, /) -> None: ...
```

- `contravariant=True` means the type can accept a superclass where a subclass is expected.
- `Sink[Garment]` can be used as `Sink[Towel]` because a sink that accepts any `Garment` can also accept a `Towel`.
- Example: `Sink[Towel]` accepts `List[Garment]` because its `append` method accepts any `Garment`, including `Towel`, but rejects `List[BathTowel]`.

| Variance | Meaning | Example |
|---|---|---|
| Covariant | A subtype can be used as its supertype | `Sequence[Shirt]` → `Sequence[Garment]` |
| Contravariant | A supertype can be used where a subtype is expected | `Sink[Garment]` → `Sink[Towel]` |
| Invariant | Cannot substitute in either direction | `MutableSequence[Shirt]` ≠ `MutableSequence[Garment]` |