#Map My Cab
This is a data engineering project at Insight Data Science. There are two goals that this project aims to accomplish:
- Provide an API for data scientists and cab dispatchers, for analyzing long term trends in cab behavior w.r.t metrics such as average pickups, dropoffs, occupancy, miles travelled etc.
- Enable a framework for real-time monitoring of cab locations, so that a user can know the unoccupied cabs across a city and zoom in on a specific neighborhood to spot and catch available cabs nearest to them.

#Data Set
Historical:
The project is based on historical geolocation data for 500 yellow cabs in San Francisco, collected over a month's time frame. The data is available as a time series, with updates on individual cab occupancy and locations at a time interval of 1 minute (approximately). The following table provides a snap shot of the raw data set (500 text files, each representing one cab):

<img src="https://github.com/PreetikaKuls/Insight-MapMyCab/blob/master/images/raw_data.png" alt="alt text" width="300" height="200">

Real-Time:
The historical data set is played back to simulate real-time behavior.

#Data Processing Framework
<img src="https://github.com/PreetikaKuls/Insight-MapMyCab/blob/master/images/pipeline.png" alt="alt text" width="600" height="300">

- Ingestion Layer (Kafka 0.8.9): The raw data is consumed by a message broker, configured in publish-subscribe mode. Each cab ID is assigned a separate key in order to preserve the temporal ordering of data for each cab. All keys are published into a common topic.
Related files: kafka_producer.py, kafka_consumer.py

- Batch Layer (HDFS, Hadoop): A kafka consumer stores the data into HDFS. Additional columns are added to the dataset to generate metrics as described in the ensuing section (1.2). This is accomplished using MrJob. Following this, tables representing the aggregate views for serving queries at the user end are generated using Hive.
Related files: Libraries: Kafka-Python

- Serving Layer (HBase): Datastore tables store the aggregate views for hour of day, day of week and individual cab profiles as generated by Hive. The table schema is optimized for quick access, by storing the hours as columns and the totals for each day in the same row (this way, hourly and daily profiles can be served from the same table/rows).
Related Files: Libraries: Happybase

- Speed Layer (Storm): The topology for processing real-time data comprises of a kafka-spout and a bolt (with tick interval frequency of 5 sec). The data is filtered to only store currently available (unoccupied) cabs into HBase. In order to serve queries with low latency, all the data is stored in one row (maximum possible columns = number of cabs - 500). 
Related Files: Libraries: Happybase, Pyleus

- Front-end (Flask): The cab locations are rendered on Google Maps and updated at 2 sec interval via AJAX. Historical data is represented as bar and line charts.
Related Files: Libraries and APIs: GoogleMaps, JQuery




Ingestion (Kafka-): 
Batch Storage: HDFS
Batch Processing: Hive, MrJob
Real-Time Streaming: Storm
Front-end: Flask

#Data Transformations
Following metrics are computed via a MapReduce operation on the raw dataset (MrJob):
- Pickup events (occupancy change 0 to 1)
- Drop off events (occupancy change 1 to 0)
- Miles travelled (based on latitude and longitude)

The resulting table is aggregated using Hive to enable batch queries such as:
- Time of day profile of pickups, dropoffs, miles travelled 
- Day of the week profile of metrics 
- Individual cab metrics (top 10 cabs with most miles travelled)

Streaming Data
- The incoming data is filtered in real-time (simulated) based on occupancy to show available cabs

 




#Live Demo:
A Live Demo of the project is available here:
Charts from sample batch queries available here:






