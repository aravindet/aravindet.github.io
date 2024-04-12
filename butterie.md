---
title: Butterie
---

A space-efficient data structure for storing variable-length strings. It is optimized for storing large sets of strings with shared prefixes in persistent storage. and supports efficient insertion, retrieval and iteration in lexical order. Based on concepts from BTrees and radix trees.

- get: O(log(N)), where N is the number of items
- getRange: O(log(N) + K), where K is the number of results in the range
- insert: O(log(N))
- delete: O(log(N))

# Overview

