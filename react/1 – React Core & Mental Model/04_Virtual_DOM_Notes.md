# Chapter 4 Notes --- Virtual DOM

## Definition

Virtual DOM is React's in-memory tree of React Elements.

## Pipeline

``` text
State
 ↓
React Elements
 ↓
Virtual DOM
 ↓
Reconciliation
 ↓
Commit
 ↓
Real DOM
```

## Key Points

-   Virtual DOM is not browser DOM.
-   React compares React Element trees.
-   Render ≠ DOM Update.
-   DOM Update ≠ Browser Paint.

## Flashcards

-   Reconciliation compares? → React Element trees.
