---
published: false
---
---
title: Clickhouse tips & tricks
date: 2017-10-29 00:00:00 Z
layout: post
---

## Query logging

Clickhouse has a inhouse query log capture mechanism. If it is activated, every query sent to Clickhouse will be saved along with valuable information about query time, duration, rows read, rows written, etc. All these information is saved in `system.query_log` table.

Activation of query logging:
Write `<log_queries>1</log_queries>`  to `/users/default` element in `users.xml`

Sample users.xml file:

    <?xml version="1.0"?>
    <yandex>
            <profiles>
              <default>
                <log_queries>1</log_queries>
              </default>
            </profiles>
            <users>
             <default>
               <profile>default</profile>
            </default>
          </users>
    </yandex>

Structure of query_log file:

|Column|Type|
|---|---|
|type | UInt8 (1,2,3,4)|
|event_date | Date |
|event_time | DateTime |
|query_start_time | DateTime |
|query_duration_ms | UInt64 |
|read_rows | UInt64 |
|read_bytes | UInt64 |
|written_rows | UInt64 |
|written_bytes | UInt64 |
|result_rows | UInt64 |
|result_bytes | UInt64 |
|memory_usage | UInt64 |
|query | String |
|exception | String |
|stack_trace | String |
|is_initial_query | UInt8 |
|user | String |
|query_id | String |
|address | FixedString(16) |
|port | UInt16 |
|initial_user | String |
|initial_query_id | String |
|initial_address | FixedString(16) |
|initial_port | UInt16 |
|interface | UInt8 |
|os_user | String |
|client_hostname | String |
|client_name | String |
|client_revision | UInt32 |
|http_method | UInt8 |

Each query is logged up to two times, on following events:

Begin of query execution (type = 1);

End of query execution (type = 2);

Exception before begin of query execution (type = 3);

Exception during query execution (type = 4);

To select slow queries, use condition on query_duration_ms column.

Query log has almost zero impact on performance.
Usually, query log is also quite compact.
