---
title: 'Clickhouse: Using disk if memory is not enough(spill to disk)'
date: 2017-10-29 22:17:00 Z
categories:
- clickhouse
tags:
- clickhouse
---

https://groups.google.com/forum/#!topic/clickhouse/cw6NRVk8miE
There is possibility to enable spilling temporary data to disk to limit memory usage for GROUP BY.
"max_bytes_before_external_group_by" setting - memory usage threshold, when temporary data will be written to filesystem. Zero (by default) means disabled.

If you are using "max_bytes_before_external_group_by", it is recommended to set "max_memory_usage" about two times larger. It is needed, because aggregation is performed in two stages: reading source data and accumulation of intermediate data (1) and merging of intermediate data (2).
Spilling data to disk is only possible at stage 1. If there was no spill, then at stage 2 it is possible, that additional amount of memory is required, about same as at stage 1.

As an example, if you has max_memory_usage = 10000000000, and you want to enable external aggregation, then it is reasonable to set max_bytes_before_external_group_by to 10000000000 and max_memory_usage to 20000000000. If there was at least one spill of temporary data, then maximum memory usage will be just little larger than max_bytes_before_external_group_by.

Sample users.xml file:
```xml
<yandex>
    <!-- Profiles of settings. -->
    <profiles>
        <!-- Default settings. -->
        <default>
            <!-- Maximum memory usage for processing single query, in bytes. -->
            <max_memory_usage>15000000000</max_memory_usag>
            <max_bytes_before_external_group_by> 7500000000 </max_bytes_before_external_group_by>
            ....
        </default>
```