# summary

The goal of this module is to provide an alternative for `system.run[...]` and Userparameters.
There are many use cases when scripts are used to query apllication metrics.
Usually they provide many metrics at the same time.
And usually such queries are quite costly.
And this is a pity that with current Zabbix data flow architecture only one value of only one metric can be returned at a time.
Storing full query result once and using it as a source for multiple item checks can significantly improve performance without sacrificing much of accuracy.
