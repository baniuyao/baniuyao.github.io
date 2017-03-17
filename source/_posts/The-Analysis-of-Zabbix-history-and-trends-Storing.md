---
title: The Analysis of Zabbix history and trends Storing
date: 2015-03-04 09:41:25
tags:
- Zabbix
- C
---

# Intro

`history` and `trends` are two main tables in Zabbix database for storing data. There are also tables inherits `history` which are used to store different types of data. For example, table `history_uint` is to store uint data. Others are `history_str`, `history_log` and `history_text`. Please pay attention that there is only one table inherits table `trends` and its name is `table_uint`.

# Entrance

The start of the whole process is in function `DCsync_all()`, `src/libs/zbxdbcache/dbcache.c`. `dbcache` is a feature which enables Zabbix to keep data in memory first and flush to database in batch.

To make it more clearly, I wrote comments inline.

``` c
static void	DCsync_all()
{
	zabbix_log(LOG_LEVEL_DEBUG, "In DCsync_all()");
	
	// call `DCsync_history` to sync history data from cache to database
	DCsync_history(ZBX_SYNC_FULL);
	// assert code is running in Zabbix Server. Zabbix Server WILL NOT run codes below
	if (0 != (daemon_type & ZBX_DAEMON_TYPE_SERVER))
		// sync trends data. 
		DCsync_trends();
	zabbix_log(LOG_LEVEL_DEBUG, "End of DCsync_all()");
}
```
# DCsync_history()

Since we are talking about Zabbix Server, so we will only take a look at `DCsync_history()`. In this paragraph, I have simplified the code in `DCsync_history()` to help readers understand the core process of flushing data to database.

``` c
DBbegin();
	
// while in Zabbix Server mode
if (0 != (daemon_type & ZBX_DAEMON_TYPE_SERVER))
{
	...
	// add data to history
	DCmass_add_history(history, history_num);
	// update trends
	DCmass_update_trends(history, history_num);
	...		
}
else
{
	DCmass_proxy_add_history(history, history_num);
	...
}
DBcommit();
```

# DCmass_add_history()

Let’s take a look into `DCmass_add_history()`.

It will calculate the numbers for each type of items.

``` c
switch (history[i].value_type)
	{
		case ITEM_VALUE_TYPE_FLOAT:
			h_num++;
			break;
		case ITEM_VALUE_TYPE_UINT64:
			huint_num++;
			break;
		case ITEM_VALUE_TYPE_STR:
			hstr_num++;
			break;
		case ITEM_VALUE_TYPE_TEXT:
			htext_num++;
			break;
		case ITEM_VALUE_TYPE_LOG:
			hlog_num++;
			break;
	}
}
```

And then, call function to write data to database.

``` c
/* history */
if (0 != h_num)
	dc_add_history_dbl(history, history_num);
/* history_uint */
if (0 != huint_num)
	dc_add_history_uint(history, history_num);
```

Finally, we can touch the sql in `dc_add_history_dbl()`.

``` c
static void	dc_add_history_dbl(ZBX_DC_HISTORY *history, int history_num)
{
	...
	for (i = 0; i < history_num; i++)
	{
		...
		zbx_db_insert_add_values(&db_insert, history[i].itemid, history[i].ts.sec, history[i].ts.ns,
				history[i].value.dbl);
	}
	...
}
```

# DCmass_update_trends

Let’s see the code first.

``` c
static void	DCmass_update_trends(ZBX_DC_HISTORY *history, int history_num)
{
	for (i = 0; i < history_num; i++)
	{
		...
		DCadd_trend(&history[i], &trends, &trends_alloc, &trends_num);
	}
	...
	while (0 < trends_num)
		// flush trends while we actually HAVE trends data to flush
		DCflush_trends(trends, &trends_num, 1);
	...
}
```

We get that the main process is in the function `DCadd_trend()`. Let’s move on to it.

``` c
static void	DCadd_trend(ZBX_DC_HISTORY *history, ZBX_DC_TREND **trends, int *trends_alloc, int *trends_num)
{
	...
	// get trend data from database by itemid
	trend = DCget_trend(history->itemid);
	...
	switch (trend->value_type)
	{
		case ITEM_VALUE_TYPE_FLOAT:
			...
			// calculate the new data
			trend->value_avg.dbl = (trend->num * trend->value_avg.dbl
				+ history->value.dbl) / (trend->num + 1);
			break;
		...
}
```

Above all, I think we are clear on the fact how Zabbix update history and trends.

Below is the process:
![Alt text](/images/zabbix_history_trends.jpg)



