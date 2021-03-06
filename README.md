# Divvy Data Challenge 2013 Toolkit

Developed by [Christopher Baker](http://github.com/bakercp) in collaboration with the [openLab](http://olab.io) at the [School of the Art Institute of Chicago](http://saic.edu).

This toolkit consists of a collection of examples, preprocessors and tools to support work on the [http://divvybikes.com/datachallenge](http://divvybikes.com/datachallenge) data set.

## Components

### DataPreprocessor

The `DataPreprocessor` is a processing sketch that makes it easy to pre-process the raw Divvy data.  The existing version removes several columns, shortens enumeration names and fixes several data anomalies found in the original raw data set.  Please see the extensive comments in the [Processing sketch](https://github.com/olab-io/divvy_datachallenge_2013_toolkit/blob/master/DataPreprocessor/DataPreprocessor.pde).  Additionally, the stations data will be keyed to the station ids in the trips file (which will result in better MySQL table normalization). 

To use the `DataPreprocesor`, place the raw data (available [here](http://divvybikes.com/assets/images/Divvy_Stations_Trips_2013.zip)) in the `data` folder of the Processing sketch.

The open the sketch in the latest version of [Processing](http://processing.org) and press the `Run` button.  The sketch will yield several "cleaned" files for use in other Processing sketches.  The current version can also be easily imported into a MySQL database for online access via the API backend. 

### API Backend Installation

While working with huge CSV files is quite possible, for online visualization and exploration, it is helpful to have a JSON compatible data api.  The preprocessed files generated by the `DataPreprocessor` can be easily imported into a MySQL database and when used with the PHP API backend, online API queries are trivial.

To set up your own Divvy data API, you will need a web server with PHP and MySQL.  Most modern servers, including shared hosting, offer this capability.  To set up the data API follow these steps:

1. Generate the `Divvy_Stations_2013_Cleaned.csv` and `Divvy_Trips_2013_Cleaned.csv` (~50MB) files using the `DataPreprocessor`.  
2. On your server, create a MySQL database called `divvy_2013` and a MySQL user called `divvy`.  The `divvy` user should have read access to the `divvy_2013` database (if this doesn't make sense, that's ok, there is an easier alternative below).
3. Next create a table called `Divvy_Stations_2013` in the `divvy_2013` database with using the following structure:

        CREATE TABLE IF NOT EXISTS `Divvy_Stations_2013` (
          `id` int(11) NOT NULL,
          `name` varchar(255) DEFAULT NULL,
          `latitude` float DEFAULT NULL,
          `longitude` float DEFAULT NULL,
          `capacity` int(11) DEFAULT NULL,
          PRIMARY KEY (`id`)
        ) ENGINE=MyISAM DEFAULT CHARSET=utf8;
Import the `Divvy_Stations_2013_Cleaned.csv` file into this table.

4. Next create a tabled called `Divvy_Trips_2013` in the `divvy_2013` database with the following structure:

        CREATE TABLE IF NOT EXISTS `Divvy_Trips_2013` (
          `trip_id` int(11) NOT NULL,
          `start_time` datetime NOT NULL,
          `stop_time` datetime NOT NULL,
          `bike_id` int(11) NOT NULL,
          `from_station_id` int(11) NOT NULL DEFAULT '-1',
          `to_station_id` int(11) NOT NULL DEFAULT '-1',
          `user_type` varchar(255) DEFAULT NULL,
          `gender` varchar(255) DEFAULT NULL,
          `birth_year` int(11) DEFAULT NULL,
          PRIMARY KEY (`trip_id`),
          KEY `from_station_id` (`from_station_id`),
          KEY `to_station_id` (`to_station_id`),
          KEY `user_type` (`user_type`),
          KEY `stop_time` (`stop_time`),
          KEY `start_time` (`start_time`),
          KEY `birth_year` (`birth_year`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
Import the `Divvy_Trips_2013_Cleaned.csv` file into this table.  _Note: The `Divvy_Trips_2013_Cleaned.csv` is quite large and it may not be possible to import the file via a simple interface like phpMyAdmin.  Instead, consider uploading the CSV file to your server, logging in via SSH and running the following command:

        mysqlimport  --ignore-lines=1 \
        --fields-terminated-by=, \
        --columns='trip_id,starttime,stoptime,bikeid,tripduration,from_station_id,from_station_name,to_station_id,to_station_name,usertype,gender,birthyear' \
        --local -u root -p divvy_2013 Divvy_Trips_2013.csv
        
5.  Next, upload [api.php](https://github.com/olab-io/divvy_datachallenge_2013_toolkit/blob/master/api/api.php) and [credentials_example.php](https://github.com/olab-io/divvy_datachallenge_2013_toolkit/blob/master/api/credentials_example.php) to the location of your choice on your server.
6.  Rename `credentials_example.php` to `credentials.php` and enter your database location and password.
7.  You are now ready to query the api using the parameters defined below.

### Data API Parameters

The API is currently a single endpoint with a simple set of parameters.  


#### Trip Start and Stop Times

Starting and stopping times are represented in UTC time inside the database.  It is the user's responsibility to convert those times back to local Chicago time.  Trip selections based on trip start and stop times can be done with the following parameters:

 Parameter         | Description                        | Example Values                        
:------------------|------------------------------------|:-----------------------------------:
 `start_min`       | _Set the start time range minimum_ |`2013-06-01` or `2013-06-01 12:00:00`     
 `start_max`       | _Set the start time range maximum_ |`2013-06-01` or `2013-06-01 12:00:00`     
 `stop_min`        | _Set the stop time range minimum_  |`2013-06-01` or `2013-06-01 12:00:00`     
 `stop_min`        | _Set the stop time range minimum_  |`2013-06-01` or `2013-06-01 12:00:00`     
 `from_station_id` | _Set the starting station id_      |_See the stations list for valid ids_     
 `to_station_id`   | _Set the ending station id_        |_See the stations list for valid ids_     
 `bike_id`         | _Set the bike id_                  |_See the trips data for valid ids_     
 `trip_id`[^1]     | _Set the trip id_                  |_See the trips data for valid ids_     
 `trip_id_min`[^1] | _Set the minimum trip id_          |_See the trips data for valid ids_     
 `trip_id_max`[^1] | _Set the maximum trip id_          |_See the trips data for valid ids_     
 `user_type`       | _Set the user type_                |`subscriber` or `customer`     
 `gender`          | _Set the user gender_              |`male` or `female`     
 `birth_year`[^2]  | _Set the user birth year_          |_Any year >= 0_     
 `birth_year_min`[^2]| _Set the minimum birth year_     |_Any year >= 0_     
 `birth_year_max`[^2]| _Set the maximum birth year_     |_Any year >= 0_     
 `age`[^2]           | _Set the user age                |_Any age >= 0_     
 `age_min`[^2]       | _Set the minimum age_            |_Any age >= 0_     
 `age_max`[^2]       | _Set the maximum age_            |_Any age >= 0_     
 `page`[^3]          | _The results page_               |_Any page >= 0_     
 `rpp`[^3]           | _Set the maximum age_            |_0 <= rpp <= 100_     
 `callback`          | _A JSONP callback_               |_Any valid javascript method name._

### Data Api Examples

For convenience the [openLab](http://olab.io) has established a public endpoint for testing.  The base endpoint URL is:

<http://data.olab.io/divvy/api.php>

All query parameter strings build from that endpoint.  If you install your own API, your endpoint URL will be different.

Select the first page of 25 results for for trips between 2013-06-01 and 2013-07-01 for males over the age of 50: 
  
  - <http://data.olab.io/divvy/api.php?start_min=2013-06-01&start_max=2013-07-01&gender=male&age_min=50&page=0&rpp=25>

To get results 26 - 50 from the same query:
  
  - <http://data.olab.io/divvy/api.php?start_min=2013-06-01&start_max=2013-07-01&gender=male&age_min=50&page=1&rpp=25>

To get all trips taken by 33 year old females:
  
  - <http://data.olab.io/divvy/api.php?gender=female&age=33>


#### Footnotes
[^1]: If a `trip_id` parameter is passed along with a `trip_id_min` and / or `trip_id_max` parameter, the `trip_id` parameter is ignored and the range style parameters are preferred.
[^2]: Both `age` and `birth_year` select on the same `birth_year` column of the database.  Since it's easier to think in terms of age, when both `age` and `birth_year` parameters are included, all `birth_year` parameters will be ignored in favor of the `age` parameter.  Like `trip_id`, the corresponding range-based versions of the `age` and `birth_year` parameters will be used.

[^3]: Since this is a massive data set, it is not advisable to let a user return huge quantities of data with a single query.  Instead, the trip results are broken down into pages of results.  `rpp` is set to 100 by default and is also the default maximum.  The `page` parameter determines which trip id to begin with.  For instance, to return results starting with the 200th trip, one might pass `rpp=100` and `page=1`. 