---
title: Simple Method to Compare to JSON in Python
date: 2016-08-25 17:20:58
tags:
- python
- json
---

# Problem

Recently I met a problem about how to compare to JSON. As we know, JSON is dict-like data structure, the order of its content is not import in the comparison. For example, `{'name': 'frank', 'age': '12'}` equals `{'age': '12', 'name': 'frank'}`. In the real world, perhaps there are some more complex data structures like list, tuple or nested-dict. 

# Solutions

In python(both python2 and python3), we can use `pprint` to get pretty print for JSON and other data structures. It will sort and format data first. And there is another method in `pprint` module called `pformat` which is used to get the output to a string. Let's see the example.

```python
import pprint

d1 = {'name': 'frank', 'age': '12'}
pformat_d1 = pprint.pformat(d1) # pformat_d1: {'age': '12', 'name': 'frank'}
d2 = {'age': '12', 'name': 'frank'}
pformat_d2 = pprint.pformat(d2) # pformat_d2: {'age': '12', 'name': 'frank'}

```