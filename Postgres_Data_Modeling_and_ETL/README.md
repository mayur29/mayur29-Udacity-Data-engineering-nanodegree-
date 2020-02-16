Sparkify Postgres ETL

This project is used to learnfollowing concepts:

1.Data modeling with Postgres
2.Database star schema created
3.ETL pipeline using Python

Background -
A startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. The analytics team is particularly interested in understanding what songs users are listening to. Currently, they don't have an easy way to query their data, which resides in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app.

Your role is to create a database schema and ETL pipeline for this analysis.

Data
Song datasets: all json files are nested in subdirectories under /data/song_data. 
    
Log datasets: all json files are nested in subdirectories under /data/log_data. 
    
Database Schema
The schema used for this exercise is the Star Schema: There is one main fact table containing all the measures associated to each event (user song plays), and 4 dimentional tables, each with a primary key that is being referenced from the fact table.


Fact Table
songplays - records in log data associated with song plays i.e. records with page NextSong

songplay_id (INT) PRIMARY KEY: ID of each user song play
start_time (DATE) NOT NULL: Timestamp of beggining of user activity
user_id (INT) NOT NULL: ID of user
level (TEXT): User level {free | paid}
song_id (TEXT) NOT NULL: ID of Song played
artist_id (TEXT) NOT NULL: ID of Artist of the song played
session_id (INT): ID of the user Session
location (TEXT): User location
user_agent (TEXT): Agent used by user to access Sparkify platform
Dimension Tables
users - users in the app

user_id (INT) PRIMARY KEY: ID of user
first_name (TEXT) NOT NULL: Name of user
last_name (TEXT) NOT NULL: Last Name of user
gender (TEXT): Gender of user {M | F}
level (TEXT): User level {free | paid}
songs - songs in music database

song_id (TEXT) PRIMARY KEY: ID of Song
title (TEXT) NOT NULL: Title of Song
artist_id (TEXT) NOT NULL: ID of song Artist
year (INT): Year of song release
duration (FLOAT) NOT NULL: Song duration in milliseconds
artists - artists in music database

artist_id (TEXT) PRIMARY KEY: ID of Artist
name (TEXT) NOT NULL: Name of Artist
location (TEXT): Name of Artist city
lattitude (FLOAT): Lattitude location of artist
longitude (FLOAT): Longitude location of artist
time - timestamps of records in songplays broken down into specific units

start_time (DATE) PRIMARY KEY: Timestamp of row
hour (INT): Hour associated to start_time
day (INT): Day associated to start_time
week (INT): Week of year associated to start_time
month (INT): Month associated to start_time
year (INT): Year associated to start_time
weekday (TEXT): Name of week day associated to start_time
    
Project structure
Files used on the project:

data folder nested at the home of the project, where all needed jsons reside.
sql_queries.py contains all your sql queries, and is imported into the files bellow.
create_tables.py drops and creates tables. You run this file to reset your tables before each time you run your ETL scripts.
test.ipynb displays the first few rows of each table to let you check your database.
etl.ipynb reads and processes a single file from song_data and log_data and loads the data into your tables.
etl.py reads and processes files from song_data and log_data and loads them into your tables.
README.md current file, provides discussion on my project.
Break down of steps followed
1. Add DROP, CREATE and INSERT query statements in sql_queries.py

2. Run in console python create_tables.py

3. Used test.ipynb Jupyter Notebook to interactively verify that all tables were created correctly.

4. Using instructions mentioned in etl.ipynb Notebook , add code to create the pipeline to process and insert all data into the tables.

5. Verfiy the changes by running test.ipynb

6. Transfer etl.ipynb notebook code to etl.py .

7. Run etl in console  python etl.py & verify the results

ETL pipeline

Prerequisites:

Database and tables created

On the etl.py we start our program by connecting to the sparkify database, and begin by processing all songs related data.

Traverse through files under /data/song_data, and for each json file encountered we use function  process_song_file to process the file.

read_json() is being used to load data in dataframe.

For each row in the dataframe we select the fields we are interested in:

song_data = [song_id, title, artist_id, year, duration]
 artist_data = [artist_id, artist_name, artist_location, artist_longitude, artist_latitude]
And finally we insert this data into their respective databases.

Once all files from song_data are read and processed, we move on processing log_data.

We repeat step 2, but this time we send our files to function process_log_file.

We load our data as a dataframe same way as with songs data.

We select rows where page = 'NextSong' only

We convert ts column where we have our start_time as timestamp in millisencs to datetime format. We obtain the parameters we need from this date (day, hour, week, etc), and insert everythin into our time dimentional table.

Next we load user data into our user table

Finally we lookup song and artist id from their tables by song name, artist name and song duration that we have on our song play data. The query used is the following:

song_select = ("""
    SELECT song_id, artists.artist_id
    FROM songs JOIN artists ON songs.artist_id = artists.artist_id
    WHERE songs.title = %s
    AND artists.name = %s
    AND songs.duration = %s
""")
The last step is inserting everything we need into our songplay fact table.