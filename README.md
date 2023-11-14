# CDF_Demo
Demonstration of the Delta Storage Format Change Data Feed

# What is the Delta Lake Change Data Feed?

The Change Data Feed is a record of changes made to a Delta Lake. 

# What purpose is served by the Change Data Feed?

The Change Data Feed can be used to solve the issue that propagating appends is straightforward, while propagating deletes or updates is a challenge. 

Enabling the Change Data Feed creates a log of changes made to the source, that can be used to propagate updates or deletes to the destination. 

# What is the format of the Change Data Feed? 

The Change Data Feed will contain the following. 

Commit Version = What Version of the Delta Lake do the changes apply to. 
update_preimage = Full record of the row before the update
update_postimage = Full record of the row after the update
Insert = Full record of the row to be inserted
delete = Full record of the row to be deleted
commit_timestamp = timestamp for the operation

# Enabling the Change Data Feed

For existing tables, 
ALTER TABLE myDeltaTable SET TBLPROPERTIES (delta.enableChangeDataFeed = true)

For all tables created in current session.
set spark.databricks.delta.properties.defaults.enableChangeDataFeed = true;

# Reading the Change Data Feed

## Static or Batch read
The following statement displays a dataframe "static_df" representing all table changes occuring in version 1-5
df will be a static, or batch dataframe. The df will not be updated if further changes occur to the source table.
static_df = spark.read.format("delta") \
  .option("readChangeFeed", "true") \
  .option("startingVersion", 1) \
  .option("endingVersion", 5) \
  .table("table_name")
display(static_df)
## Streaming read
The following statement displays a streaming dataframe "streaming_df" representing all table changes that have occured to table "table_name" since version 3.

The dataframe streaming_df will be a streaming dataframe. The dataframe when displayed will update when new versions of the table "table_name" are created by append, delete or update operations. 

streaming_df = spark.readStream.format("delta") \
  .option("readChangeFeed", "true") \
  .option("startingVersion", 3) \
  .table("table_name")
display(streaming_df)

# Applying Changes from the Change Data Feed to a destination

Delta Lake supports merge statements. 

A merge statement is a sql statement that takes a collection of input rows, compares them to existing rows in the destination and using conditional logic can apply a combination of inserts, updates or deletes.

Use of the merge statement in either batch operations, or as part of a foreachbatch operation in streaming is typically how the read from the Change Data Feed is applied to a downstream destination. 

  
