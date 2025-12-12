---
Stage: 
Start Date: 
Release Date: Unreleased
Release Versions:
  ember-source: vX.Y.Z
  ember-data: vX.Y.Z
Relevant Team(s): 
RFC PR: 
---

<!--- 
Directions for above: 

Stage: Leave as is
Start Date: Fill in with today's date, YYYY-MM-DD
Release Date: Leave as is
Release Versions: Leave as is
Relevant Team(s): Fill this in with the [team(s)](README.md#relevant-teams) to which this RFC applies
RFC PR: Fill this in with the URL for the Proposal RFC PR
-->

# Glimmer 2: Types

## Summary

This RFC describes a way to document components in a way to connect it to tooling from the wider javascript ecosystem. The scope of this RFC is to cover documentation for **arguments**, **named blocks/yield** and **element(s)** for components written in JavaScript, TypeScript and template-only.

## Motivation

> Why are we doing this? What use cases does it support? What is the expected
outcome?

What we want do:

- Types for Documentation
- Types for Tool Integration (e.g. glint/uELS/template-lint)
- Applicable from TS/Template-Only/JS

## Detailed design

### Definitions

At first let's define terms that will be used throughout this RFC:

- **Block**
  For any block invocation of a component, this describes the contents in there. We can distinguish between _default_ block and named blocks.
  This is defined in syntax section for [RFC 460 - Yieldable Named Blocks](https://emberjs.github.io/rfcs/0460-yieldable-named-blocks.html#syntax)
  
  - _Default Block_
    That is the default block when invoking a component:

    ```hbs
    <Modal>
      this is the default block
    </Modal>
    ```

    equivalent to:

    ```hbs
    <Modal>
      <:default>this is the default block</:default>
    </Modal>
    ```

  - _Named Block_
    Named blocks can be referred to via an identifier and are represented through `<:identifier>` syntax:

    ```hbs
    <Modal>
      <:header>Named header block</:header>
      <:body>Named body block</:body>
    </Modal>
    ```

- **Block Param**
  Parameters passed from the component (through `{{yield}}`) to the consumer:
  
  ```hbs
  <Modal as |close|>
    <button {{on "click" close}}>close</button>
  </Modal>
  ```
  
  In the example `close` is a parameter _to_ the block. On the consuming side a space delimited list of parameters can be accessed within the pipe syntax (`| .. |`).
  
- **Yield**
  Yielding is how a component invokes a provided block, optionally passing it parameters. [Example from RFC 460](https://emberjs.github.io/rfcs/0460-yieldable-named-blocks.html#block-parameters):
  
  ```hbs
  <article>
    <header>{{yield @article.title to='header'}}</header>
    <section>{{yield @article.body to='body'}}</section>
  </article>
  ```

### Signature Interface

```ts
interface ComponentSiganture {
  Element?: HTMLElement | HTMLElement[];
  Arguments?: Record<string, unknown>;
  BlocksOrYields?: {
    default?: [...blockParams: unknown[]];
    [name: string]?: [...blockParams: unknown[]];
  }
}
```

- Question: How are we going to document multiple elements? A tuple? An array?
- Question: are we going to use `Arguments` or `Args`?
- Question: Blocks or Yields?
  - Authors: Yield to blocks with optional params
  - Consumers: Using blocks and their optional params
  - TG: For me personally, the types/docs are written from authors _for_
    consumers. As a matter of that, it should be _read_ from consumers
    perspective and for that is `Blocks` then.

### Type Utilities

Are we going to ship `ComponentLike`, `ComponentWithBoundArgs` and maybe
`ArgsFor` from glint?

## How we teach this

> What names and terminology work best for these concepts and why? How is this
idea best presented? As a continuation of existing Ember patterns, or as a
wholly new one?

> Would the acceptance of this proposal mean the Ember guides must be
re-organized or altered? Does it change how Ember is taught to new users
at any level?

> How should this feature be introduced and taught to existing Ember
users?

### Migration from Glimmer1 to Glimmer2

Under the hood we can auto-transform the types from glimmer1 to glimmer2, that
way we can built migration into the types itself and no immediate action is
needed. [See the PoC by Dan](https://tsplay.dev/wQAY7w).

- Question: Will there be some linting, etc. in place to hint at this
  deprecation?
- Question 2: Will there be a codemod to assist transforming this?
  - Will it be useful to have such a codemod? It will turn an object into a
    sub-`Args` object of the signature. By manually changing it, you can at the
    same time add types for blocks and elements?

## Drawbacks

> Why should we *not* do this? Please consider the impact on teaching Ember,
on the integration of this feature with other existing and planned features,
on the impact of the API churn on existing apps, etc.

> There are tradeoffs to choosing any path, please attempt to identify them here.

## Alternatives

> What other designs have been considered? What is the impact of not doing this?

> This section could also include prior art, that is, how other frameworks in the same domain have solved this problem.
>

## Unresolved questions

> Optional, but suggested for first drafts. What parts of the design are still
TBD?

### What About `Attributes`?

Initially we also thought about documenting html **attributes** for a component. While we investigated, what a possible `attributes` key on the type signature object would _mean_. We identified two scenarios:

1. The `attributes` would describe what attributes that component would already provide, so you as a consumer don't have to - or if you are unhappy are free to overwrite them. With the added benefit, that tools such as template-lint can lint against _expected_ final element and attributes, e.g. to check for `aria-*` completeness.
2. The `attributes` would offer a list of possible attributes to pass in to enrich the experience with the component at hand

Both cases are valid, yet to much unknowns/experience for us to clearly say this is the right way to document either. However, given the object for the type signature, we can add this later in a non-breaking way while at the same time gain experience and allow the ember community to experiment in this area.
