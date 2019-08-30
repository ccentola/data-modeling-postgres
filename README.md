# Data Modeling with Postgres
## Authors
Carl Centola | [github](https://github.com/ccentola) | [linkedin](https://www.linkedin.com/in/carlcentola/) |
___

## Overview
Sparkify, a music streaming startup, is looking for a data engineer to create a database schema and ETL pipline to assist the company in analyzing user data. In particular, the company is interested in understanding what songs users are listening to.

The objective of this project is to ingest the data provided by Sparkify and design a method of storage using a relational data model in Postgres.

### Relational Model Reasoning
* The data types are structured (we know before-hand the sctructure of the jsons we need to analyze, and where and how to extract and transform each field)
* Ability to use SQL is necesary to gain insights from the data we store.
* As of the present, the size of the data is not big enough to justify a NoSQL solution.
___
## Data
**song data** - metadata about a song and the artist of that song. Song data files are partitioned by the first 3 letters of each track's song id. For example:
```
song_data/A/B/C/TRABCEI128F424C983.json
song_data/A/A/B/TRAABJL12903CDCF1A.json
```
The following is sample data from a single song:

```json
{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, "artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", "title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0}
```
**log data** - simulation of activity logs from Sparkify app. Log files are partitioned by year and month:
```
log_data/2018/11/2018-11-12-events.json
log_data/2018/11/2018-11-13-events.json
```
The following is sample data from a single log file:
```json
{"artist":"The Grass Roots","auth":"Logged In","firstName":"Sara","gender":"F","itemInSession":72,"lastName":"Johnson","length":166.71302,"level":"paid","location":"Winston-Salem, NC","method":"PUT","page":"NextSong","registration":1540809153796.0,"sessionId":411,"song":"Let's Live For Today","status":200,"ts":1542153802796,"userAgent":"\"Mozilla\/5.0 (iPhone; CPU iPhone OS 7_1_2 like Mac OS X) AppleWebKit\/537.51.2 (KHTML, like Gecko) Version\/7.0 Mobile\/11D257 Safari\/9537.53\"","userId":"95"}
```
___
## Database Design
This project implements a star schema.  `songplays` is the fact table in the data model, while `users`,`songs`,`artists`, and `time` are all dimensional tables.

### Fact Table
`songplays` - records in log data associated with page = "NextSong".
* `songplay_id`, `user_id`, `level`, `song_id`, `artist_id`, `session_id`, `location`, `user_agent`

### Dimension Tables
`users` - collection of app users.
* `user_id`, `first_name`, `last_name`, `gender`, `level`

`songs` - collection of songs.
* `song_id`, `title`, `artist_id`, `year`, `duration`


`artists` - information about artists.
* `artist_id`, `artist_name`, `artist_location`, `artist_latitude`, `artisit_longitude`


`time` - timestamps of records in songplays deconstructed into various date-time parts.
* `start_time`, `hour`, `day`, `week`, `month`, `year`, `weekday`

___
## Repository Structure
* `data` folder - includes `song_data` and `log_data`; raw jason files provided by Sparkify.
* `etl.ipynb` - environment for designing and testing ETL process.
* `etl.py` - contains all ETL procedures.
* `create_tables.py` - creates the database and tables.
* `sql_queries.py` - contains DDL/DML required for the ETL process.
* `test.ipynb` - runs a series of test queries to ensure that tables are present and data is flowing through the ETL process as expected.
___
## ETL Pipeline
At a high level, the ETL process extracts the data specified in `create_tables.py` from the song and log data.

### Procedure
1. Establish a connection to the Sparkify database.
2. Pass appropriate data and helper function into the `process_data` function.
   1. Process song files in `/data/song_data` using the `process_song_file` function.
   2. Process log files in `/data/log_data` using the `process_log_file` function.
3. Close the connection.

#### `process_song_file`
A function that, given a connection cursor and a filepath, extracts data elements from the song data files and inserts them using `song_table_insert` and `artist_table_insert` queries.

#### `process_log_file`
A function that, given a connection cursor and a filepath, generates the fact table (`songplays`) by extracting timestamp and user data, then inserting the cleaned data into their coreresponding tables via `time_table_insert` and `user_table_insert`, respectively.
___
## How to Run
1. Run `create_tables.py`. This **must** be the first script run, as it creates the tables that the ETL process depends on.
2. Test that tables are created using `test.ipynb`.
3. Run `etl.py` to process data files and insert values into tables.
4. Test that values have been inserted correctly using `test.ipynb`

