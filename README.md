# Poker Hand Classifier — Two Approaches


A 5-card poker hand classifier (cards 2–9, no face cards) implemented two ways as a study in how your choice of data structure changes what the problem *is*.

---

## Version 1: Sort-then-scan (`PokerHandsFinal.cpp`)

The straightforward approach. Sort the hand with bubble sort, then check for hands by looking at adjacency patterns in the sorted array.

**Key insight:** Sorting lets you reduce every hand-detection function to a few index comparisons. Consecutive equal values = a match group.

**Gotcha I found out after:** The `if/else if` chain in `main()` has to be ordered carefully. Four-of-a-kind before full house before three-of-a-kind etc. — because the sorted array checks aren't mutually exclusive on their own. The *order of evaluation* is doing work that the detection logic doesn't.

---

## Version 2: Union-Find (PokerHandsUF.cpp)

Instead of sorting and scanning, we model "same value" as a graph connectivity problem.

- Each card is a node (indices 0–4)
- If two cards share a value, **union** their nodes
- After building components, extract the **partition** — the multiset of component sizes
- Classify based solely on the partition shape


| Hand | Partition (sorted desc) |
|---|---|
| Four of a Kind | `[4, 1]` |
| Full House | `[3, 2]` |
| Three of a Kind | `[3, 1, 1]` |
| Two Pair | `[2, 2, 1]` |
| One Pair | `[2, 1, 1, 1]` |
| High Card | `[1, 1, 1, 1, 1]` |

The `classify()` function reads like a truth table. No ordering dependency — the partition is an unambiguous fingerprint.

Implementation uses **weighted union by size** + **path compression** on a fixed 5-element array. O(α(5)) ≈ O(1) effectively.

 **Note:** Straight detection isn't included in the UF version — straights are about *order*, not *equivalence classes*, so they don't map naturally to union-find. The sort-based version handles this correctly.

---

## What I Learned

The sort version works great and the code is simple. But the if/else ordering issue exposed that it's implicitly relying on evaluation order to resolve ambiguity — that's hidden logic.

The union-find version makes the structure explicit: a hand *is* a partition, and classification is just pattern matching on that partition. Cleaner separation of concerns, easier to extend (add new hand types without worrying about chain order).

Two valid solutions to the same problem with genuinely different tradeoffs.

---

## Build

```bash
g++ PokerHandsFinal.cpp -o poker_sort
g++ PokerHandsUF.cpp -o poker_uf
---
