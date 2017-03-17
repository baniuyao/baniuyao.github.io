title: Tricks on Crontab
date: 2015-12-09 18:01:40
tags:
- Linux
- Crontab
---

# '@' in Crontab

'@' sysmbol is available in crontab and it very useful especially for `@reboot`. `@reboot` makes it possible to run a script while machine starts.

``` bash
@reboot     -   This runs the Cron job when the machine is started up or if the Cron daemon is restarted

@midnight   -   This runs the Cron job once a day at midnight, it's the equivalent of 0 0 * * *

@daily      -   Does exactly the same as @midnight

@weekly     -   This runs a Cron job once a week on a Sunday the equivalent of 0 0 * * 0

@monthly    -   This runs a Cron job once a month on the first day of every month at midnight and is the same as 0 0 1 * *

@annually   -   Runs a Cron job once a year at midnight on the first day of the first month and is the equivalent of 0 0  1 1 *

@yearly     -   The same as annually
```

An example for run a command when the machine starts:

``` bash
@reboot date >> /tmp/date.log
```
