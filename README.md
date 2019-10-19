# Sparkify song play logs ETL process

The objective of this project is to create a SQL analytics database for a fictional music streaming service called Sparkify. Sparkify's analytics team seeks to understand what, when and how users are playing songs on the company's music app. The analysts need an easy way to query and analyze the songplay data, which is currently stored in raw JSON logs and metadata files on a local directory.

I have implemented an ETL pipeline in python to process and upload the data into a PostgreSQL database. The ETL process extracts each songplay from the list of page actions recorded by the app. Data for analysis, such as song name, user information, subscription tier, and location of user, is structured into the main songplay table and related dimensional tables.

### Data
- **Song datasets**: all json files are nested in subdirectories under */data/song_data*. A sample of this files is:

```
{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, "artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", "title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0}
```

- **Log datasets**: all json files are nested in subdirectories under */data/log_data*. A sample of a single row of each files is:

```
{"artist":"Slipknot","auth":"Logged In","firstName":"Aiden","gender":"M","itemInSession":0,"lastName":"Ramirez","length":192.57424,"level":"paid","location":"New York-Newark-Jersey City, NY-NJ-PA","method":"PUT","page":"NextSong","registration":1540283578796.0,"sessionId":19,"song":"Opium Of The People (Album Version)","status":200,"ts":1541639510796,"userAgent":"\"Mozilla\/5.0 (Windows NT 6.1) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/36.0.1985.143 Safari\/537.36\"","userId":"20"}
```

## Database Schema

The sparkify database design uses the simple star schema shown below. The schema contains one fact table, songplays, and four dimension tables: songs, artists, users and time. The fact table references the primary keys of each dimention table, enabling joins to songplays on song_id, artist_id, user_id and start_time, respectively. This structure will enable the analysts to aggregate the data efficiently and explore it using standard SQL queries.

On why to use a relational database for this case:
- The data types are structured (we know before-hand the sctructure of the jsons we need to analyze, and where and how to extract and transform each field)
- The amount of data we need to analyze is not big enough to require big data related solutions.
- Ability to use SQL that is more than enough for this kind of analysis
- Data needed to answer business questions can be modeled using simple ERD models
- We need to use JOINS for this scenario

![DataBase Structure](https://github.com/Abhinavkaitha/Insight_DE/blob/master/output/ER.png)

### Song Plays table

- *Name:* `songplays`
- *Type:* Fact table

| Column | Type | Description |
| ------ | ---- | ----------- |
| `songplay_id` | `INTEGER` | The main identification of the table | 
| `start_time` | `DATE` | The timestamp that this song play log happened |
| `user_id` | `INTEGER NOT NULL REFERENCES users (user_id)` | The user id that triggered this song play log. It cannot be null, as we don't have song play logs without being triggered by an user.  |
| `level` | `TEXT` | The level of the user that triggered this song play log |
| `song_id` | `TEXT REFERENCES songs (song_id)` | The identification of the song that was played. It can be null.  |
| `artist_id` | `TEXT REFERENCES artists (artist_id)` | The identification of the artist of the song that was played. |
| `session_id` | `INTEGER` | The session_id of the user on the app |
| `location` | `TEXT` | The location where this song play log was triggered  |
| `user_agent` | `TEXT` | The user agent of our app |

### Users table

- *Name:* `users`
- *Type:* Dimension table

| Column | Type | Description |
| ------ | ---- | ----------- |
| `user_id` | `INTEGER PRIMARY KEY` | The main identification of an user |
| `first_name` | `TEXT NOT NULL` | First name of the user, can not be null. It is the basic information we have from the user|
| `last_name` | `TEXT NOT NULL` | Last name of the user. |
| `gender` | `TEXT` | The gender is stated with just one character `M` (male) or `F` (female). Otherwise it can be stated as `NULL` |
| `level` | `TEXT` | The level stands for the user app plans (`premium` or `free`) |


### Songs table

- *Name:* `songs`
- *Type:* Dimension table

| Column | Type | Description |
| ------ | ---- | ----------- |
| `song_id` | `TEXT PRIMARY KEY` | The main identification of a song | 
| `title` | `TEXT NOT NULL` | The title of the song. It can not be null, as it is the basic information we have about a song.|
| `artist_id` | `TEXT NOT NULL REFERENCES artists (artist_id)` | The artist id, it can not be null as we don't have songs without an artist, and this field also references the artists table. |
| `year` | `INTEGER` | The year that this song was made |
| `duration` | `FLOAT NOT NULL` | The duration of the song |


### Artists table

- *Name:* `artists`
- *Type:* Dimension table

| Column | Type | Description |
| ------ | ---- | ----------- |
| `artist_id` | `TEXT PRIMARY KEY` | The main identification of an artist |
| `name` | `TEXT NOT NULL` | The name of the artist |
| `location` | `TEXT` | The location where the artist are from |
| `latitude` | `FLOAT` | The latitude of the location that the artist are from |
| `longitude` | `FLOAT` | The longitude of the location that the artist are from |

### Time table

- *Name:* `time`
- *Type:* Dimension table

| Column | Type | Description |
| ------ | ---- | ----------- |
| `start_time` | `DATE PRIMARY KEY` | The timestamp itself, serves as the main identification of this table |
| `hour` | `INTEGER` | The hour from the timestamp  |
| `day` | `INTEGER` | The day of the month from the timestamp |
| `week` | `INTEGER` | The week of the year from the timestamp |
| `month` | `INTEGER` | The month of the year from the timestamp |
| `year` | `INTEGER` | The year from the timestamp |
| `weekday` | `INTEGER` | The week day from the timestamp |

## The project file structure

We have a small list of files, easy to maintain and understand:
 - `sql_queries.py` -  query repository to use throughout the ETL process
 - `create_tables.py` - file creates the schema 
 - `etl.py` -  file responsible for the main ETL process
 - `etl.ipynb` - this notebook that is used to develop the logic behind the `etl.py` process
 - `test.ipynb` -  this notebook can be used to check if our ETL process was being successful (or not).
 
 ## Instructions on how to run the program
 
  ```
python create_tables.py
```
```
python etl.py
```
