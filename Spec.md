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
--------------------|--------------------------------------------------
`id`                | unique limited length identifier of the `command`
`command`           | shell command or a script _which must return JSON_
`maxage`            | maximum age of a cached JSON which will qualify as valid source of information
`path`,`to`,`metric`| path to a part of JSON which should be extracted

# implementation

In `zbx_module_init()` (before Zabbix forks) module will create a shared memory segment where it will store a cache index and a semaphore to manage concurrent access to it from various Zabbix processes. In the index module will store entries consisting of:
* command `id`;
* timestamp of the last JSON update;
* id of a separate shared memory segment where JSON is stored.

> It is possible to use `command` as a unique key instead of `id`. The idea behind `id` is to store in index as many entries as possible while keeping its size under control and not limiting `command` length. As a positive side effect, `command` can be modified without losing cached results. As a negative side effect, user will have to deal with potential `id` conflicts (especially when agent is polled by multiple servers).

When module performs item check it first checks for the presense of `id` in cache index. If `id` is not represented in cache, module runs `command`, stores JSON in a separate shared memory segment and updates index. If `id` is present in cache but the latest JSON is more than `maxage` old, module runs `command` and updates stored JSON and index. If `id` is present in cache and JSON is not older than `maxage`, module takes currently stored JSON for further processing.

Module uses parameters after `maxage` as keys to locate a value needs to extract. If the value is a string, it should be unquoted. If the value is boolean, numeric, JSON object or JSON array, it should be returned as is.

Module must respect timeout configured for all item checks in Zabbix configuration file.

In `zbx_module_uninit()` module must mark all shared memory segments for destruction and remove semaphore.

# things to discuss

* fixed length `id` to minimize shared memory segment for cache index and unlimited `command`,  __vs.__ no `id`, moderate index size and limited length `command` __vs.__ no `id`, oversized index and unlimited `command`
* ability to extract values from arrays within JSON
* other `command` output formats to support
