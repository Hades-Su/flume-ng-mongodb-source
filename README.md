# flume-ng-mongodb-source

This project is used for [flume-ng](https://github.com/apache/flume) to communicate with mongodb databases

-------------------------------

Compilation and packaging
----------
```
  $ mvn clean install
```

Deployment
----------

Copy flume-ng-mongodb-source-[version].jar in target folder into flume lib dir folder
```
  $ cp target/flume-ng-mongodb-source-1.0-SNAPSHOT.jar $FLUME_HOME/lib
```

Configuration of SQL Source:
----------

| Property Name | Default | Description |
| :-----------: | :-----: | :---------: |
| <b>type</b> | org.hades.flume.source.MongoDBSource | The component type name, needs to be org.hades.flume.source.MongoDBSource  |
| host | localhost | db host |
| port | 27017 | db port |
| db | events | db name |
| collection | events | db collection |
| statusPath | /var/lib/flume/mongodbSource.status | Path to save the status file. It is used to recover from the breakpoint of the field "_id", but at the current stage, only recovery by a single incremental type (preferably ObjectId type) is supported. |
| batchSize | 100 | Used in limit. The max number of lines to read and send to the channel at a time |
| useIdField | true | true or false. Indicates the use of the field "_id". The default value is true, which means that the query results will include the field. Otherwise, it means that the field will be filtered and the query result will not contain the value of the field. |
| pollInterval | 1000 | Time (in milliseconds), which is the interval between each query. |
| authenticationEnabled | false | true means login by username/password, false means login without authentication |
| username | - | required when "authenticationEnabled" is true |
| password | - | required when "authenticationEnabled" is true |

Configuration example
--------------------

```properties
agent.sources = r1
agent.channels = c1
agent.sinks = k1

agent.sources.r1.channels = c1
agent.sinks.k1.channel = c1

agent.sources.r1.type = org.hades.flume.source.MongoDBSource
agent.sources.r1.host = localhost
agent.sources.r1.port = 27017
agent.sources.r1.model = SINGLE
# agent.sources.r1.db = events
# agent.sources.r1.collection = events
# agent.sources.r1.batchSize = 100
# agent.sources.r1.useIdField = false
# agent.sources.r1.statusPath = /var/lib/flume/mongodbSource.status
# agent.sources.r1.pollInterval = 1000
# agent.sources.r1.authenticationEnabled = false

agent.channels.c1.type = memory
agent.channels.c1.capacity = 1000
agent.channels.c1.transactionCapacity = 100

agent.sinks.k1.type = file_roll
agent.sinks.k1.sink.directory = /var/lib/flume
agent.sinks.k1.sink.rollInterval = 0
```

How to avoid duplicate queries and lost data?
--------------------

This source uses the field "_id" in mongodb to implement breakpoint recovery. The first query will generate a status file that saves the lastIndex. This ensures that the second query can start from the position of lastIndex.

But because the field "_id" can store multiple types, the query statement will miss data, lead to the current source only supports breakpoints of a single incremental type, preferably the ObjectId type, because other types will miss data, such as I have "_id ": 100,200, then the lastIndex saved in the status file generated during the first query is 200. If add" _id ": 10 in this time, it will to query $ gt: 200 for the second query, and 10 is less than 200. Was queried and lost data 10.
