# Change Data Capture @ MySQL

## **Objective**

Capture changes in a MySQL Database Table (INSERT , UPDATE , DELETE)  In A Chronological Order .

### How Was This Done Historically&#x20;

Traditional System relied on a periodic process which includes a dump of the MySQL table and send it to downstream data stores which can then check for difference. There are several drawbacks of this process:

* Data incompleteness, all data changes are not captured. If the dump is taken once in a day then we won’t be able to record the intermediate state for records which were updated more than once in a day.
* No Real Time , Runs As A Batch Process

### The main advantages of CDC are:

* CDC typically captures changes in real-time, keeping downstream systems, such as data warehouses, always up-to-date and enabling event-driven data pipelines.
* Using CDC decreases the load on all involved systems since only relevant information, i.e., data change events, are processed.
* CDC enables straightforward implementations of use cases requiring access to data changes, such as an audit or changelog, without changing the code of applications.\


### Push vs Pull

There are two main ways for change data capture systems to operate. Either the source system pushes changes to the target, or the target periodically polls the source and pulls the changed data.

**Push-based** systems often require more work for the source system, as they need to implement a solution that understands when changes are made and send those changes in a way that the target can receive and action them. The target system simply needs to listen out for changes and apply them instead of constantly polling the source and keeping track of what it's already captured. This approach often leads to lower latency between the source and target because as soon as the change is made the target is notified and can action it immediately, instead of polling for changes.

The downside of the push-based approach is that if the target system is down or not listening for changes for whatever reason, they will miss changes. To mitigate this, queue- based systems are implemented in between the source and the target so that the source can post changes to the queue and the target reads from the queue at its own pace. If the target needs to stop listening to the queue, as long as it remembers where it was in the queue it can stop and restart where it left off without missing any changes.

**Pull-based** systems are often a lot simpler for the source system as they often require logging that a change has happened, usually by updating a column on the table. The target system is then responsible for pulling the changed data by requesting anything that it believes has changed.

The benefit of this is the same as the queue-based approach mentioned previously, in that if the target ever encounters an issue, because it's keeping track of what it's already pulled, it can restart and pick up where it left off without any issues.

The downside of the pull approach is that it often increases latency. This is because the target has to poll the source system for updates rather than being told when something has changed. This often leads to data being pulled in batches anywhere from large batches pulled once a day to lots of small batches pulled frequently.

![](<../../.gitbook/assets/Screenshot 2022-02-17 at 12.08.05 PM.png>)

The rule of thumb is that if you are looking to build a real-time data processing system then the push approach should be used. If latency isn’t a big issue and you need to transfer a high volume of bulk updates, then pull-based systems should be considered.

There are  common approaches to implementing change data capture: triggers, queries, and MySQL’s Binlog , Debezium .&#x20;

## Approach One&#x20;

### Change Data Capture with Triggers in MySQL

#### Using Triggers And Auxiliary Table

![](<../../.gitbook/assets/Screenshot 2022-02-17 at 12.55.45 AM.png>)

### Triggers

Triggers we can create a trigger, which is an event that is fired when an INSERT, UPDATE, DELETE occurs on a given table.

### Auxiliary Table&#x20;

We can have an auxiliary table that has a row inserted for each INSERT/UPDATE on the main table, AFTER it happens containing the ID of the row changed, the old value the new value and a timestamp.

Using MySQL’s support for [TRIGGER functions](https://dev.mysql.com/doc/refman/8.0/en/trigger-syntax.html) we could listen for all INSERT, UPDATE, and DELETE events occurring in the table of interest and, for each event, insert one row into another table, effectively building a changelog.

#### Example

Let’s assume a MySQL database features the following table _users_:

```
CREATE TABLE users (
  user_id    INT,
  user_name  TEXT,
  user_email TEXT
);
```

Example: Schema of the MySQL table _users_

We could create a trigger, which captures all INSERT events from the table _users_ and appends a new row for each INSERT event to the table _users\_change\_events_:

```
CREATE TABLE users_change_events (
  log_id          BIGINT AUTO_INCREMENT,
  event_type      TEXT,
  event_timestamp TIMESTAMP
  user_id         INT,
  user_name       TEXT,
  user_email      TEXT
  PRIMARY KEY ('log_id')
);

CREATE TRIGGER user_insert_capture
 AFTER INSERT ON users
 FOR EACH ROW BEGIN
 INSERT INTO users_change_events (
   event_type,
   event_timestamp,
   user_id,
   user_name,
   user_email
 )
 VALUES (
   'INSERT',
   now(),
   NEW.user_id,
   NEW.user_name,
   NEW.user_email
 );
```

Example: Schema of the MySQL table _users\_change\_events_ and definition of the MySQL trigger _user\_insert\_capture_

UPDATE and DELETE events can be captured accordingly.

By default, this approach stores captured events only inside MySQL. If you want to sync change events to other data systems, you would have to recurrently query the MySQL table holding the events, which increases the complexity.

#### Pros

Let us have a look at the pros and cons of implementing change data capture with triggers in MySQL:

* Changes are captured instantly, enabling the real-time processing of change events.
* Triggers can capture all event types: INSERTs, UPDATEs, and DELETEs.
* Using triggers, it's easy to add custom metadata to the change event, such as the name of the database user performing the operation.

#### Cons

* Triggers increase the execution time of the original operation and thus hurt the performance of the database.
* Triggers require changes to the MySQL database.
* Changes to the schema of the monitored table (here _users_) must be manually propagated to the table filled by the trigger function (here _users\_change\_events_).
* If change events shall be synced to a data store other than the same MySQL database, we would need to set up a data pipeline, which polls the table filled by the trigger function (here _users\_change\_events_).
* Creating and managing triggers induces additional operational complexity.

Triggers are not always the best way to record change events in a database. As triggers can get invalidated once the schema of the table changes, and it would in turn cause the actual table operation to fail.



## Approach Two

## Change Data Capture with Binlog in MySQL&#x20;

### Binary Log or Binlog&#x20;

The binary log is a set of log files that contain information about data modifications made to a MySQL server instance. There are two types of binary logging:&#x20;

* **Statement-based logging** : Events contain SQL statements that produce data changes (inserts, updates, deletes)
* **Row-based logging** : Events describe changes to individual rows. Available from MySQL 5.1 onwards.

&#x20;MySQL offers the [Binlog](https://dev.mysql.com/doc/refman/8.0/en/binary-log.html) for efficiently and safely replicating data between different database instances on possibly different physical machines. Technically, the Binlog is a binary file on disk, which holds all events that change the content or structure of the MySQL database, e.g., INSERTs, UPDATEs, DELETEs, schema changes, etc.

For the purpose of implementing change data capture, we might attach to the Binlog of the MySQL database and consume all change events occurring in the monitored table.

### **Configurations changes at MySQL**

While many database systems already use the Binlog for replication purposes it is not enabled by default. If your MySQL instance does not yet use the Binlog, you can enable it by introducing the following changes to the MySQL configuration:

```systemd
server-id         = 42
log_bin           = mysql-bin
binlog_format     = ROW
expire_logs_days  = 10
# define `binlog_row_image` for MySQL 5.6 or higher,
# leave it out for earlier releases
binlog_row_image  = FULL
```

&#x20;                                   ****                                    Enable the Binlog in the configuration of MySQL.

&#x20;A MySQL user must be defined that has all of the following permissions on all of the databases that the connector will monitor:&#x20;

```sql
SELECT RELOAD SHOW DATABASES REPLICATION SLAVE REPLICATION CLIENT
```

Sample command:&#x20;

```sql
GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, 
REPLICATION CLIENT ON . TO 'debezium' IDENTIFIED BY 'dbz';
```

More information around the configuration changes can be found on the following link: [https://debezium.io/documentation/reference/1.0/connectors/mysql.html#setting-up-mysql](https://debezium.io/documentation/reference/1.0/connectors/mysql.html#setting-up-mysql)



Managed MySQL instances typically do not directly expose access to the configuration. However, most managed MySQL services, which we are aware of, offer support for the MySQL Binlog, e.g., AWS RDS, Google Cloud SQL, or Azure Database.

The following list shows the advantages and disadvantages of using MySQL’s Binlog for implementing CDC:

* Binlog-based CDC enables the event-driven capturing of data changes in real-time. Downstream applications have always access to the latest data from MySQL.
* Binlog-based CDC can detect all change event types: INSERTs, UPDATEs, DELETES, and even schema changes.
* Since reading events from the Binlog boils down to directly accessing the file system, this CDC approach does not impact the performance of the MySQL database.
* The Binlog is not available in very old versions of MySQL (at DataCater we support MySQL 5.5 or newer).
* The Binlog does not store the entire history of change events performed on the database table but only the operations performed within a particular retention period (defined by the configuration option _expire\_logs\_days_), which is why we typically combine it with an initial full snapshot of the monitored table using a _SELECT \* FROM table\_name;_ query.

### Binlog-based change data capture can be Implemented In Various Ways:

### Using CDC (Change Data Capture) systems like Debezium with Kafka , Kafka Connect.

### Kafka Connect&#x20;

Kafka Connect is a framework for connecting Kafka with external systems such as databases, search indexes, and file systems. Connectors are used for copying data between Kafka and another system.&#x20;

Connectors come in two flavors:&#x20;

&#x20;          **Source Connectors**  :    Import data from another system to Kafka.&#x20;

&#x20;          **Sink Connectors**        :    Export data from Kafka to another system.

Connectors do not perform any data copying themselves. It is responsible for breaking down the job into a set of Tasks.

**Task**: Copies subset of data to or from Kafka. It also has two types:&#x20;

* Source Task&#x20;
* Sink Task

![](https://lh5.googleusercontent.com/c6oEHRGTh07KN7crfF9oZ3uLa-\_nBpQlHhMgreGBjwSym6J4RYZaWInkcbqma3oRBKWCMOyy4j7dNVWFjqrydKaj-ltZXUwb2owW62bdcHKpRtRdY-3W8x9HkLJkGrun6WYAi9-2)

### Debezium&#x20;

Debezium is a set of distributed services to capture changes in your databases so that your applications can see those changes and respond to them. Debezium records all row-level changes within each database table in a change event stream, and applications simply read these streams to see the change events in the same order in which they occurred. Source: https://debezium.io/documentation/reference/1.0/index.html

Debezium provides a low latency CDC platform for multiple databases like MySQL, Postgres using Apache Kafka. We are using Debezium’s MySQL connector for recording all the row level changes in the databases on MySQL server/cluster. It also provides the capability to take a snapshot of the existing data.

### **Integration**

Debezium’s MySQL connector can be added as a source connector with Kafka Connect. Once configured, it pulls the data/events from MySQL by reading binlog in near real-time and pushes them in a Kafka topic. Users can then consume by writing Kafka Consumer for the kafka Topic or can write a Kafka sink connector and promptly respond to DB modification events.

![Without Kafka Connect](<../../.gitbook/assets/Screenshot 2022-02-17 at 12.07.47 PM.png>)

![With Kafka Connect](<../../.gitbook/assets/Screenshot 2022-02-17 at 12.13.00 PM.png>)

### Understanding format of the data written to Kafka

&#x20;Each record has two parts schema and payload. The schema key contains the schema for the row, txn record, source etc. The payload key contains the data change. The payload key also contains the before and after for the row in case the row was updated or deleted. It also contains the operation type which is useful for knowing the event type that is inserted, updated or deleted. Example of a payload written to the Kafka corresponding to the update change event by Debezium connector. Source: [Debezium documentation](https://debezium.io/documentation/reference/1.0/connectors/mysql.html#reading-the-binlog)

![](https://lh5.googleusercontent.com/2jrpRvC762rMeLBmJv6tRHrel6olyH7HcvVShPbgolPT8FuW7thZVVBWPQwR7egxT3pEbV3vkkU-bCDJ2w62PLl19uGiFGMsg0KxtXta0BJ8g2eMh6VbqC8Ovp3RtOBR45q8zYCc)

### Consuming Binlog  **Using  Open Source CDC (Change Data Capture) systems like**  [Debezium](https://debezium.io) or [Maxwell’s daemon](https://maxwells-daemon.io)

Sometimes we may not want to rely upon an external cluster of Kafka brokers and Kafka Connect services  . Instead, we can embed Debezium connectors directly within the application space which will respond to DB changes directly . We still want the same data change events, but prefer to have the connectors send them directly to the application rather than persist them inside Kafka.

This can reduce Infrastructure Cost + Development time significantly.

#### Integration

To use the Debezium Engine module, add the debezium-api module to your application’s dependencies. There is one out-of-the-box implementation of this API in a debezium-embedded module which should be added to the dependencies too. For Maven, this entails adding the following to your application’s POM:

**Maven**

![](https://lh3.googleusercontent.com/P9S4U2FV3Icq5AgSiCWZ5BbNElEuGBZeQHbgEHEbVN2Io7HJhzhnR9egVxQjKFPYWuLgB9sjkonKXgGB02sOrBskpXrl8T7s\_yCtkSqediWYd-4DewvVVk9zWcJUzwXJQsVuXPBT)

Likewise, add dependencies for each of the Debezium connectors that your application will use. For example, the following can be added to your application’s Maven POM file so your application can use the MySQL connector:

![](https://lh4.googleusercontent.com/qobEKkRZUklasyOPU-oqXXcZcKc4mJZ0EHCk3g-uYlTLf49s7salzVzw8TU1QAtbAPG-kp4vggnJ3N9OOqll\_-RpL7Zm4fpTtprQuMXSg1Qw9ZX9uCHywREdMgQTa31ZwegDOY4X)

#### __[_Code Changes To Capture And Integrate Embedded Debezium_ ](https://debezium.io/documentation/reference/stable/development/engine.html)__

### **References**

* ****[**https://reorchestrate.com/posts/debezium-performance-impact/**](https://reorchestrate.com/posts/debezium-performance-impact/)
* [https://debezium.io/documentation/reference/1.0/connectors/mysql.html#reading-the-binlog](https://debezium.io/documentation/reference/1.0/connectors/mysql.html#reading-the-binlog)
* [https://datacater.io/blog/2021-08-25/mysql-cdc-complete-guide.html](https://datacater.io/blog/2021-08-25/mysql-cdc-complete-guide.html)
* [https://stackoverflow.com/questions/15357483/how-to-create-triggers-to-add-the-change-events-into-audit-log-tables](https://stackoverflow.com/questions/15357483/how-to-create-triggers-to-add-the-change-events-into-audit-log-tables)
* [https://rockset.com/blog/cdc-mysql-postgres/](https://rockset.com/blog/cdc-mysql-postgres/)
* [https://rockset.com/blog/change-data-capture-what-it-is-and-how-to-use-it](https://rockset.com/blog/change-data-capture-what-it-is-and-how-to-use-it)

[****\
****](https://reorchestrate.com/posts/debezium-performance-impact/)****

### &#x20;****&#x20;
