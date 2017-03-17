---
title: Connect MySQL with ssh tunnel in Python
date: 2016-07-04 14:45:41
tags:
- Python
- SSHTunnel
- MySQL
---

Sometimes you can only connect to MySQL via a certain server for security reasons. But apparently, coding in IDE(e.g PyCharm) is easier. For this circumstance, we can use [sshtunnel](https://pypi.python.org/pypi/sshtunnel).

So we have a MySQL server on `127.0.0.3:3306`, its mysql login name and password is `MYSQL_USER` and `MYSQL_PASSWORD`. And we can only connect to this mysql via server `127.0.0.2:22`, and its login name and password is `root` and `SERVER_PASSWORD`. Let's figure out how `sshtunnel` is working.

``` python
 with SSHTunnelForwarder(('127.0.0.2', 22), ssh_password='SERVER_PASSWORD', ssh_username='root', remote_bind_address=('127.0.0.3', 3306)) as server:
        conn = MySQLdb.connect(host='127.0.0.1', port=server.local_bind_port, user='MYSQL_USER', passwd='MYSQL_PASSWORD')
        cursor = conn.cursor()
```
