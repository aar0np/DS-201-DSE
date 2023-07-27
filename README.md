<!-- TOP -->
<div class="top">
  <img class="scenario-academy-logo" src="https://datastax-academy.github.io/katapod-shared-assets/images/ds-academy-2023.svg" />
</div>

# Exercises for DS201

## Table of Contents

1. [Install and Start Cassandra](#1-install-and-start-cassandra)
2. [Quick Wins](#2-quick-wins)
3. [Partitions](#3-partitions)
4. [Clustering Columns](#4-clustering-columns)
5. [Application Connectivity](#5-application-connectivity)

## Start GitPod instance

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/aar0np/DS-201-DSE)

## Exercises

### 1. Install and Start Cassandra

Click the GitPod button above.  It should start a new instance for you.

Your GitPod instance should have DSE 6.8.372 already downloaded.

✅ View the downloaded tarball:
```
ls -l
```

✅ Extract tarball:
```
tar -zvxf dse-6.8.tar.gz
```

The directory should now contain the tarball and the `dse-6.8.32` directory

✅ View the directory:
```
ls -l
```

✅ Switch to the `dse-6.8.32` directory.
```
cd dse-6.8.32
```

✅ View the `bin` directory:
```
ls -l bin
```

✅ Start DSE:
```
bin/dse cassandra
```

It may take a while for DSE to start.  While it is displaying output in the terminal, it is running in the background.
Feel free to press any key once the output stops moving.

✅ Run *nodetool* to determine Cassandra's status (you may have to run this command multiple times until Cassandra starts):
```
bin/nodetool status
```
---
### 2. Quick Wins

#### Create a keyspace and a table

Welcome to the KillrVideo company! KillrVideo hired you to build the latest and greatest video sharing application on the Internet. Your task is to ramp up on the domain and become acquainted with Apache Cassandra. To start, you are looking into creating a table schema and to load some data.

The video metadata is made up of:

<table class="katapod-table">
  <tr>
    <th>Column Name</th>
    <th>Date Type </th>
  </tr>
  <tr>
    <td>video_id</td>
    <td>timeuuid</td>
  <tr>
  <tr>
    <td>added_date</td>
    <td>timestamp</td>
  <tr>
    <tr>
    <td>title</td>
    <td>text</td>
  <tr>
</table>

✅ Use `nodetool` to verify that Cassandra is running (you may need to run this multiple times):
```
bin/nodetool status
```

✅ Start the command line tool `cqlsh`:
```
bin/cqlsh
```

- Or -
```
bin/cqlsh 127.0.0.1 -u cassandra -p cassandra
```


✅ Create a keyspace called *killrvideo*. Use `NetworkTopologyStrategy` for the replication class with a data center of "Cassandra" and a replication factor of one.

```
CREATE KEYSPACE killrvideo
WITH replication = {
  'class':'NetworkTopologyStrategy', 
  'Cassandra': 1
};
```

✅ Switch to the newly created keyspace with the *USE* command:

```
USE killrvideo;
```

✅ Create a table called `videos` with columns `video_id` of type `TIMEUUID`, `added_date` of type `TIMESTAMP` and `title` of type `TEXT`. Designate `video_id` as the primary key.

```
CREATE TABLE videos (
  video_id TIMEUUID,
  added_date TIMESTAMP,
  title TEXT,
  PRIMARY KEY (video_id)
);
```

#### Insert and Load Data

<table class="katapod-table">
  <tr>
    <th>video_id</th>
    <th>added_date</th>
    <th>title</th>
  </tr>
  <tr>
    <td>36b8bac0-6260-11ea-ac4c-87a8af4b7ed0</td>
    <td>2020-03-09</td>
    <td>Foundations of DataStax Enterprise</td>
  <tr>
</table>

```
INSERT INTO videos (video_id, added_date, title)
VALUES (36b8bac0-6260-11ea-ac4c-87a8af4b7ed0, '2020-03-09', 'Foundations of DataStax Enterprise');
```

✅ Use a `SELECT` statement to retrieve your row from the table.

```
SELECT * from videos;
```

Remember though, using a `WHERE` clause is a good habit to get into:

```
SELECT * FROM videos
WHERE video_id=36b8bac0-6260-11ea-ac4c-87a8af4b7ed0;
```

✅ Insert another row into the table.

```
INSERT INTO videos (video_id, added_date, title)
VALUES (95fe9800-2c2f-11b2-8080-808080808080, '2020-01-20', 'Cassandra Data Modeling');
```

✅ Retrieve all rows from the table.
```
SELECT * from videos LIMIT 10;
```

✅ Delete all previously inserted rows from the table using the `TRUNCATE` statement and verify that the table is empty.

```
TRUNCATE videos;
SELECT * from videos;
```

✅ Use the `COPY` command to import data into your `videos` table.
```
COPY videos(video_id, added_date, title)
FROM '/workspace/DS-201-DSE/data/videos.csv'
WITH HEADER=TRUE;
```

✅ Retrieve all rows from the table to verify that the table loaded correctly.
```
SELECT * from videos LIMIT 10;
```
---
### 3. Partitions

#### Explore the `videos` table

✅ Verify that Cassandra is running.
```
bin/nodetool status
```
✅ Start 'cqlsh' so you can execute CQL statements:
```
bin/cqlsh
```
✅ Switch to the killrvideo keyspace via the USE command:
```
USE killrvideo;
```

✅ Execute the following command to view information about the videos table:
```
DESCRIBE TABLE videos;
```

##### Question: What is the partition key for the videos table?

##### Question: How many partitions are in the videos table?

---
**Note:** This table has a single row in each partition rather than using partitions to group related rows. This is an anti-pattern!

---

✅ Execute the following query to see how partition key values are mapped to tokens by the partitioner:
```
SELECT token(video_id), video_id FROM videos LIMIT 10;
```

✅ Exit *cqlsh*:
```
quit
```
- Or -

```
exit
```

✅ Inspect the `videos-by-tag.csv` file that we will import into a new table:
```
cat /workspace/DS-201-DSE/data/videos-by-tag.csv
```

- Or -
```
gp open /workspace/DS-201-DSE/videos-by-tag.csv
```

- Or just click on it in GitPod. -

Notice how this CSV file categorizes the videos using *tags*: `datastax`, and `cassandra`.

✅ Start *cqlsh* again and switch to the *killrvideo* keyspace:

```
cqlsh

USE killrvideo;
```

✅ Create a new table with name `videos_by_tag` that can store data from file *videos_by_tag.csv*, such that we get one partition for every tag and each partition can have multiple rows with videos. 

```
CREATE TABLE videos_by_tag (
  tag TEXT,
  video_id TIMEUUID,
  added_date TIMESTAMP,
  title TEXT,
  PRIMARY KEY ((tag), video_id)
);
```

✅ Use the COPY command to import the `videos-by-tag.csv` data into your new table:
```
COPY videos_by_tag (tag, video_id, added_date, title)
FROM '/workspace/DS-201-DSE/data/videos-by-tag.csv'
WITH HEADER = TRUE;
```

✅ Retrieve all rows from table *videos_by_tag* and verify that you get 5 rows as expected.

If the table only has 2 rows instead of 5, make sure that the table primary key has enough columns so that it can uniquely identify each row.

You can always `TRUNCATE videos_by_tag`, then execute `DROP TABLE videos_by_tag;` and go back to the previous steps to make any necessary corrections.

```
SELECT * FROM videos_by_tag LIMIT 10;
```

#### Run some queries

✅ Retrieve all videos tagged with cassandra from table `videos_by_tag`
```
SELECT * FROM videos_by_tag WHERE tag = 'cassandra';
```

✅ Retrieve all videos tagged with datastax from table `videos_by_tag`
```
SELECT * FROM videos_by_tag WHERE tag = 'datastax';
```

✅ Finally, retrieve any video titled Cassandra & SSDs from table videos_by_tag.
```
SELECT * FROM videos_by_tag WHERE title = 'Cassandra & SSDs';
```

You should see an error something like this. This is expected. Cassandra only allows efficient queries on primary key columns, which for this table doesn’t include the `title` column.

> InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"


✅ For now, you can use ALLOW FILTERING to execute this query but should know that this is an anti-pattern! The query requires scanning all rows in the table, which is not feasible for real-life large data sets.
```
SELECT * FROM videos_by_tag 
WHERE title = 'Cassandra & SSDs' ALLOW FILTERING;
```
---
### 4. Clustering Columns

Clustering columns are the columns that are part of the primary key, but are not part of the partition key. This exercise will help you understand how clustering columns affects queries and how you can filter rows with them.

✅ Verify that Cassandra is running.
```
bin/nodetool status
```

✅ Start 'cqlsh' so you can execute CQL statements:
```
bin/cqlsh 127.0.0.1 -u cassandra -p cassandra
```
✅ Switch to the killrvideo keyspace via the USE command:
```
USE killrvideo;
```

✅ Execute the following command to view information about the videos table:
```
DESCRIBE TABLE videos_by_tag;
```

You should see the following table definition
```
CREATE TABLE killrvideo.videos_by_tag (
    tag text,
    video_id timeuuid,
    title text,
    PRIMARY KEY (tag, video_id)
) ...
```

In this table the prartition key is tag and the clustering column is video_id. Therefore, rows are grouped by tag and ordered by video_id.

Clustering columns support both equality and inequality predicates for CQL queries and also allow ordering within a partition. Define and execute several CQL queries against table videos_by_tag that use equality (=) and inequality (>,>=,<,<=) predicates, as well as row ordering with the ORDER BY clause.

✅ Select all videos:
```
SELECT * FROM videos_by_tag LIMIT 10;
```

✅ Select a specific partition:
```
SELECT * FROM videos_by_tag
WHERE tag = 'cassandra';
```

✅ Select a specific video:
```
SELECT * FROM videos_by_tag
WHERE tag = 'cassandra' AND
      video_id =  245e8024-14bd-11e5-9743-8238356b7e32;
```

✅ Select videos using an inequality:
```
SELECT * FROM videos_by_tag
WHERE tag = 'cassandra' AND
      video_id <= 245e8024-14bd-11e5-9743-8238356b7e32;
```

✅ Select videos using an inequality and reverse the order:
```
SELECT * FROM videos_by_tag
WHERE tag = 'cassandra' AND
      video_id <= 245e8024-14bd-11e5-9743-8238356b7e32
ORDER BY video_id DESC;
```

#### Create the latest_videos_by_tag table.

In this portion of the exercise, you will create a table that supports queries like this one.

```
SELECT tag, video_id, added_date, title
    FROM latest_videos_by_tag
    WHERE tag = 'cassandra'
    ORDER BY added_date DESC;
```
Rows will be grouped by tag and ordered by added_date. You will need to use `video_id` as part of the primary key to ensure uniqueness.

✅ Create the table:
```
CREATE TABLE latest_videos_by_tag (
  tag TEXT,
  video_id TIMEUUID,
  added_date TIMESTAMP,
  title TEXT,
  PRIMARY KEY ((tag), added_date, video_id)
) WITH CLUSTERING ORDER BY (added_date DESC, video_id ASC);
```

✅ Import videos-by-tag.csv into the new table:
```
COPY latest_videos_by_tag(tag, video_id, added_date, title)
FROM '/workspace/DS-201-DSE/data/videos-by-tag.csv'
WITH HEADER = TRUE;
```

✅ Retrieve all the rowa from latest_videos_by_tag:
```
SELECT * FROM latest_videos_by_tag LIMIT 10;
```

Verify that you get 4 rows as expected.

The rows should be grouped by a tag and, within each partition, ordered in descending order of an added date.

✅ Execute the original CQL query that table latest_videos_by_tag was designed for:
```
SELECT tag, video_id, added_date, title
FROM latest_videos_by_tag
WHERE tag = 'cassandra'
ORDER BY added_date DESC;
```

✅ Change the original query below to return the cassandra videos added after 2013-02-01:
```
SELECT tag, video_id, added_date, title
FROM latest_videos_by_tag
WHERE tag = 'cassandra' AND
      added_date > '2013-02-01'
ORDER BY added_date DESC;
```

✅ Change the original query below to return the oldest cassandra video from the table. Use LIMIT 1 to return only the first video in a result set:
```
SELECT tag, video_id, added_date, title
FROM latest_videos_by_tag
WHERE tag = 'cassandra'
ORDER BY added_date ASC
LIMIT 1;
```
---
### 5. Application Connectivity

- [Workshop code w/ examples for Python, and Java](https://github.com/datastaxdevs/workshop-cassandra-application-development/)
- Simple examples for Go, Python, and Java:	
	- [testCassandra.go](https://github.com/aar0np/go_stuff/blob/main/testCassandra.go)
	- [testCassandra.py](https://github.com/aar0np/DS_Python_stuff/blob/main/testCassandra.py)
	- [TestCassandra.java](https://github.com/aar0np/testcassandra/blob/main/src/main/java/testcassandra/TestCassandra.java)

- [Awesome Astra](https://awesome-astra.github.io/docs/pages/develop/)

---

Link to Java exercise repo: [DS-Java-DSE](https://github.com/aar0np/DS-Java-DSE/tree/main)

