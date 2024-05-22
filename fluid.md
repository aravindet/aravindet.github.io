---
title: Fluid Layout
layout: default
---

# Introduction

CSS has several layout modes. Per <a href='https://developer.mozilla.org/en-US/docs/Web/CSS/Layout_mode' target='_blank'>MDN</a>:

- Normal flow (block and inline)
- Float layout
- Table layout
- Positioned layout
- Multi-column layout
- Flexible box layout
- Grid layout

This proposal introduces Fluid Layout, a set of low-level layout primitives that can succinctly represent any layout expressible using the existing CSS layout modes. It can be considered a simpler and more general version of Grid layout.

The generality allows it to represent layouts that are not possible using CSS, such as "masonry-style" layouts.

The goal is to create an (1) engine that implements Fluid Layout, and (2) a set of transformations that can convert any valid CSS layout into the equivalent Fluid Layout.

# Overview

The Fluid Layout engine accepts a tree of nodes (like the DOM) and computes, for each node, the coordinates of the rectangle it should should occupy on the screen.

Each node might have certain properties that influence how it is rendered. Two sets of properties are relevant to Fluid Layout:

**flow** on a parent node controls how its children are positioned within it.

**place** on a child node controls how it is placed and sized relative to its parent and neighbours.

# Flow

The flow property may have the value 'yield' if it does not create a new layout context but allows its children to be laid out along its parent's layout context.

Otherwise, it specifies a list of axes, which are directions along which successive items are placed.

Most typical layouts have two axes. For example, in text layout the first axis runs left to right, and the second axis runs top to bottom. However, flexbox (without wrapping) has only one axis while multi-column layout has three axes: words run from left to right, lines run top to bottom, and columns run left to right again. Layouts with even more axes are possible with Fluid.

An axis may specify a list of tracks which are positions along that axis where children may be placed. Tracks may be created automatically to fit the content and available space, just like in Grid layout.

Tracks of an axis may cross those of other axes to form grids. Each axis may specify crosses, a list of other axes (identified by index, starting with 1) that its tracks cross.

For example, a wrapping normal layout has two axes. The first axis does not cross the second (words on separate lines do not align vertically), but the second one crosses the first (all words on a line align horizontally).

## Definition

```
flow: 'yield' | [ {
  direction: 'fore' | 'back' | 'left' | 'right' | 'up' | 'down'

  tracks: [ {
    min: length | 'auto'
    max: length | 'auto'
    gap: boolean
    repeat: integer | 'auto'
  } ]

  crosses: [ integer ]
} ]
```

# Place

By default, children are placed in the same order as they appear in the tree. This can be overridden using the order property, just like in Flex layout. The default value is 'auto'. If order is 'none', the item will be placed in the earliest available flow position; this is similar to the dense option in Grid layout.

Children may be placed on specific tracks of some or all the axes, using the start, end and span properties. These are tuples with one element for each axis, and are analogous to CSS properties like grid-row-start. A negative value for start and end is counted from the end of the axis; -1 is the last track on that axis.

The align property, also a tuple, controls how the child should align with the tracks on each axis.

The position property renders the child outside its normal location in the flow, like with CSS positions absolute, fixed, relative and sticky. It also specifies float behavior.

The wrap_before and wrap_after properties control whether this item (or the succeeding item) advances to the next track on each axis. For each axis, it may be 'never', 'always' or a wrapping cost represented by a unitless number.

## Definition

```
place: {
  order: 'auto' | integer
  axes: [{
    size: integer | length
    start: integer | length
    end: integer | length
    align: 'start' | 'end' | number
    wrap_before: 'never' | 'always' | number
    wrap_after: 'never' | 'always' | number
  }]
}
```

```
length = {
  px: number
  em: number
  pc: number
  fr: number
}
```

# Questions

Tracks can only be auto-spawned in the wrap direction.
No: tables - requires explicit wrap points
How will auto-spawn in the middle work? Will it only work in dense layout?
Can same track be auto-spawn and auto-fill/fit?
What does none/none mean? Rectangle packing?
