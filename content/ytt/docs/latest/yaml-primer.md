---
title: "How ytt Annotates YAML"
---

## Overview

As noted in [Why Structural Templating?](../../../blog/why-structural-templating.md), templates in `ytt` are well-formed YAML files with templating annotated on, rather than YAML-like text files with templating inserted in.

That article details the benefits of a structure-based approach. It also notes that many of us have reasonable expectations of how to write templates because we've been doing so for years. But once the template ceases to be a simple text file and instead is a YAML document set, some of those expectations are foiled.

Spending a few minutes reviewing:

- [what YAML is](#a-yaml-refresher), and 
- [how `ytt` annotates YAML](#annotating-yaml)
  
can save hours of frustration.

## A YAML Refresher

YAML is a tree of nodes of these types:

- **Document Set** (which is group of **Documents**)
- **Map** that has **Map Items**
- **Array** that has **Array Items**

### Documents and Document Sets

A file containing YAML is parsed into a **Document Set**, which is nothing more than a collection of **Document**s.

![a file containing two YAML documents](/images/ytt/yaml-primer/two-docs.jpg)
_Figure 1: A file, parsed into a **document set** (dotted blue) containing two **documents** (solid blue)._

Note: `---` marks the start of a new document.

The corresponding tree of nodes looks something like this:

![](/images/ytt/yaml-primer/two-docs-ast.jpg)
_Figure 2: Corresponding tree: this document set has two documents._

A given document has exactly one (1) value. It is either:
- a **Map**,
- an **Array**, or
- a **Scalar**

We'll look at each of these, in turn. Maps are most common...

### Maps and Map Items

A "map" is collection of key-value pairs. Each pair is referred to as a "map item".

![](/images/ytt/yaml-primer/two-maps-and-items.jpg)
_Figure 3: Each document has a **map** (dotted green) each containing two **map items** (solid green)._

The complete tree of nodes looks like this:

![](/images/ytt/yaml-primer/two-maps-and-items-ast.jpg)
_Figure 4: Corresponding tree: each document has a map for its value; each map has two items._

Let's zoom in on the first document, and explicitly reveal the contents of a map item: 

![](/images/ytt/yaml-primer/map-items-ast.jpg)
_Figure 5: Each map item has a **key** and a **value**; here, both map items' key is a string and their value is an integer._

Like documents, a map item has exactly one (1) value. And just like documents, it's either:
- a **Map**,
- an **Array**, or
- a **Scalar**

Let's see at what it looks like for a map item to have a map, itself.

![](/images/ytt/yaml-primer/map-item-has-map-ast.jpg)
_Figure 6: The document's value is a map with two map items; the second map item has a key of "metadata" and a value that is another map._


### Arrays and Array Items

An "array" is a list of "array items" (zero-indexed).

Like documents (and map items), an array item has exactly one (1) value. Once more, it's either:
- a **Map**
- an **Array**, or
- a **Scalar**

![](/images/ytt/yaml-primer/map-item-has-array-ast.jpg)
_Figure 7: The **map item** "foo" has an **array** (dotted gold) for a value; that array has three items (solid gold)._

### Scalars

A "scalar" is a fundamental value. YAML supports:
- integers (e.g. 13, 42, 137, 32767)
- floating point numbers (e.g. 1.0, 3.14, 137.03599913)
- strings (e.g. "", "fine structure", "$ecr3t")
- boolean values: `true` or `false`
- `null` (when a value is omitted, that implies `null`; `~` == `null`)

### Summary

In short:

- a YAML file is parsed into a **Document Set** which is merely a list of **Document**s.
- a given **Document** has one _value_.
- Both **Map**s and **Array**s are nothing more than collections of items.
  - a **Map Item** has a _key_ and a _value_
  - an **Array Item** has a _value_
- a _value_ (whether held by a document, map item, or array item) is either a **Map**, an **Array**, or a Scalar.

With this structure in hand, we can now look into how `ytt` annotates it.

## Annotating YAML

A `ytt` "annotation" is a YAML comment of a specific format that attaches some behavior to a node.

For example:

![](/images/ytt/yaml-primer/map-item-with-value.jpg)
_Figure 8: A YAML file with a `ytt` annotation._

Is understood to mean that the value of `foo` is the result of the expression `13 + 23 + 6`.

Note that this template is still well-formed YAML.

The diagram reveals how the annotation is attached to the YAML tree:

![](/images/ytt/yaml-primer/map-item-ann-with-value-ast.jpg)
_Figure 9: The annotation (dotted black) is attached to the **map item** (solid green)._

This is a document, whose value is a map, that contains a single map item. To the right of the map item is an annotation that contains an expression that, when evaluated, becomes a _value_. The annotation is attached to the map item.

### How an Annotation Finds its Node

When attaching annotations, `ytt` follows these two rules (in order):
1. the annotation is attached to the value-holding node **on its left**, if there is one; otherwise,
2. the annotation is attached to the value-holding node **directly below** it.

In the example above, the value-holding node to the left of the annotation is the map item.

To be explicit, "value-holding" nodes are:
- **Document**s,
- **Map Item**s, and
- **Array Item**s.

_(Note these items are illustrated using solid lines rather than dotted.)_

Let's see these rules at play:

![](/images/ytt/yaml-primer/overlay-ann-on-doc-and-map.jpg)
_Figure 10: A templated document with three annotations._

We'll ignore what these annotations mean, for now, and focus on where they attach. There are three annotations in total. Visualizing the YAML tree can help:


![](/images/ytt/yaml-primer/overlay-ann-on-doc-and-map-ast.jpg)
_Figure 11: Each annotation attaches to the value-holding node to its left or bottom._

Taking each annotation in turn:
- `@overlay/match by=overlay.all`:
  - is in the document set, but that is not a value-holding node
  - has a **document** node just below it and so is attached to _that_ node.
- `@overlay/replace`:
  - has no node to its left;
  - has a **map** just below it but maps are _not_ value-holding;
  - the next node is a **map item**, and so attaches to _that_ node.
- `@template/value 13 + 23 + 6`:
  - has a **map item** to its left, and so attaches to _that_ node.
  - there _is_ another **map item** below it (i.e. `bar: true`), but a home has already been found for this annotation.



## Further Reading

- put your understanding to the test by picking an example from [the Playground](https://carvel.dev/ytt/#example:example-demo) and tweaking it in various ways to see what _actually_ happens.
- if you'd like to obsess over YAML itself, look no farther than the [YAML spec](https://yaml.org/spec/1.2/spec.html).



---

## Advanced Topics

The following are topics that may do more to confuse than help those just getting start with `ytt`. We recommend saving the sections below until there's a need to dig deeper into this subject.

### How to Annotate a Map Item contained in an Array Item

As one starts authoring more precise overlays, a common scenario shows up that seems 
a situation arises for which the solution may not be obvious. This is the case where one wishes to annotate a **map item** that's in an **array item**.


### Viewing a file's Abstract Syntax Tree (AST)

The tree diagrams shown on this page are very similar to the actual data structures `ytt` handles internally. In fact, you can ask the tool to render it.

Given `foo.yml`:

```yaml
#@overlay/match by=overlay.all
---
#@overlay/replace
foo: #@ 13+23+6
bar: true
```

Invoke `ytt` normally, and include the `--debug` flag:

```console
$ ytt -f foo.yml --debug
...
### ast
????: docset (obj=0xc0001804d0)
       1: doc (obj=0xc0000b5080)
        : <nil>
       2: doc (obj=0xc0000b5200)
    meta:    1: '@overlay/match by=overlay.all'
           2: map (obj=0xc0000b50e0)
           4: key=foo (obj=0xc0000b5140)
        meta:    3: '@overlay/replace'
        meta:    4: '@ 13+23+6'
            : <nil>
           5: key=bar (obj=0xc0000b51a0)
            : true
...
```

Scroll to the section titled `### ast` and you'll find the abstract syntax tree (AST) described.

Here are some highlights:

- our **document** — with the first `@overlay` annotation attached — whose value is a **map**:
    ```console
    ...
           2: doc (obj=0xc0000b5200)
        meta:    1: '@overlay/match by=overlay.all'
               2: map (obj=0xc0000b50e0)
    ...
    ```
- that **map**'s first **map item** has a key of `foo`, initial value of `null` and two annotations attached (noticed annotations are listed as "`meta`", immediately underneath the node):
    ```console
    ...
               2: map (obj=0xc0000b50e0)
               4: key=foo (obj=0xc0000b5140)
            meta:    3: '@overlay/replace'
            meta:    4: '@ 13+23+6'
                : <nil>
    ...
    ```
- finally the second **map item** has a key of `bar` and a value of `true`:
    ```console
    ...
               5: key=bar (obj=0xc0000b51a0)
                : true
    ...
    ```

### Translation of `ytt` terms to the YAML Specification

(Some of the terms here differ from those used in the [YAML spec](https://yaml.org/spec/1.2/spec.html). We use _these_ terms because they are more common and are the same used in `ytt` itself.
