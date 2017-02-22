# summary

The goal of this module is to provide an alternative for `system.run[...]`, UserParameters and external checks.
There are many use cases when scripts are used to query apllication metrics.
Usually they provide many metrics at the same time.
And usually such queries are quite costly.
And this is a pity that with current Zabbix data flow architecture only one value of only one metric can be returned at a time.
Storing full query result once and using it as a source for multiple item checks can significantly improve performance without sacrificing much of accuracy.

# user interface

Module will aim to provide just one item key.

__`eco.run[id,command,maxage,path,to,metric]`__

parameter           | meaning
--------------------|---------------------------------
`id`                | unique identifier of the command
`command`           | shell command or a script _which must return JSON_
`maxage`            | maximum age of a cached JSON which will qualify as valid source of information
`path`,`to`,`metric`| path to a part of JSON which should be extracted
