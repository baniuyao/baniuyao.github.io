---
title: Using pip and easy_install with Chinese repo
date: 2016-06-20 15:47:16
tags:
- pip
- easy_install
- python
- pypi
---

For some reasons, `pypi.python.org` is not available or has a slow download rate. We can simply using [douban pypi](http://pypi.douban.com/simple) or [v2ex pypi](http://pypi.v2ex.com/simple) by two ways.

## edit env

Edit env will enable `pip` using own pypi in default. In Linux and Mac, the file is in `%HOME/.pip/pip.conf`. And the content is:

```
[global]
find-links=http://pypi.douban.com/simple
[install]
find-links=
    http://pypi.douban.com/simple
    http://pypi.v2ex.com/simple
```