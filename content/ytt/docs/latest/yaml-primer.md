---
title: "YAML Refresher"
---

## Overview

As noted in [Why Structural Templating?](../../../blog/why-structural-templating.md), 

Tools

`ytt`, however
`ytt` operates _within_ the structure



## Summary

YAML is a tree of nodes of a specific set of types:

- **Document Set** (which is group of **Documents**)
- **Map** that has **Map Items**
- **Array** that has **Array Items**
- **Scalars**

### Documents and Document Sets

A file containing YAML is parsed into a **Document Set**, which is nothing more than a collection of **Document**s.

![a file containing two YAML documents](/images/ytt/yaml-primer/two-docs.jpg)
_Figure 1: A file, parsed into a document set containing two documents._

Note: `---` marks the start of a new document.

The corresponding tree of nodes looks something like this:

![](/images/ytt/yaml-primer/two-docs-ast.jpg)
_Figure 2: Corresponding tree: a document set has two documents._


