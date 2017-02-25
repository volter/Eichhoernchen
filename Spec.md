# summary

The goal of this module is to provide an alternative for `system.run[...]`, UserParameters and external checks.
There are many use cases when scripts are used to query apllication metrics.
Usually they provide many metrics at the same time.
And usually such queries are quite costly.
And this is a pity that with current Zabbix data flow architecture only one value of only one metric can be returned at a time.
Storing full query result once and using it as a source for multiple item checks can significantly improve performance without sacrificing much of accuracy.

# user interface

Module will aim to provide just one item key.

__`eco.run[command,maxage,path,to,metric]`__

parameter           | meaning
--------------------|---------------------------------------------------
`command`           | shell command or a script _which must return JSON_
`maxage`            | maximum age of a cached JSON which will qualify as valid source of information
`path`,`to`,`metric`| path to a part of JSON which should be extracted

# implementation

In `zbx_module_init()` (before Zabbix forks) module will create a shared memory segment where it will store a cache index and a semaphore to manage concurrent access to it from various Zabbix processes. In the index module will store entries consisting of:
* `command`;
* timestamp of the last JSON update;
* id of a separate shared memory segment where JSON is stored.

When module performs item check it first checks for the presense of `command` in cache index. If `command` is not represented in cache, module runs `command`, stores JSON in a separate shared memory segment and updates index. If `command` is present in cache but the latest JSON is more than `maxage` old, module runs `command` and updates stored JSON and index. If `command` is present in cache and JSON is not older than `maxage`, module takes currently stored JSON for further processing.

Module uses parameters after `maxage` as keys to locate a value needs to extract. If the value is a string, it should be unquoted. If the value is boolean, numeric, JSON object or JSON array, it should be returned as is.

Module must respect timeout configured for all item checks in Zabbix configuration file.

In `zbx_module_uninit()` module must mark all shared memory segments for destruction and remove semaphore.

# things to discuss

* ability to extract values from arrays within JSON
* other `command` output formats to support

# decisions

* abandoned an idea of fixed length `id` for each `command`
