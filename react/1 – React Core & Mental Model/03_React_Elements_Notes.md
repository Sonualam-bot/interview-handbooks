# Chapter 3 Notes --- React Elements

## Definition

A React Element is an immutable JavaScript object describing UI.

## JSX Pipeline

``` text
JSX
 ↓
Compiler
 ↓
React.createElement()/jsx()
 ↓
React Element
```

## Example

``` js
{ type:'button', props:{ children:'Click'} }
```

## Key Points

-   JSX is not HTML.
-   Components return React Elements.
-   React Elements are immutable.

## Flashcards

-   React.createElement returns? → React Element
