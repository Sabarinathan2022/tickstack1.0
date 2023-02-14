# Custom TICK stack Sandbox 2.0
This is a custom TICK stack sandbox with:  


TELEGRAF 1.2
INFLUXDB 2.1
CHRONOGRAF 1.9
KAPACITOR 1.6
GRAFANA 7.1.0 

### How to Run Sandbox

To run the `sandbox`, simply use the convenient cli:

```bash
$ ./sandbox
sandbox commands:
  up           -> spin up the sandbox environment (add -nightly to grab the latest nightly builds of InfluxDB and Chronograf)
  down         -> tear down the sandbox environment
  restart      -> restart the sandbox
  influxdb     -> attach to the influx cli
  flux         -> attach to the flux REPL

  enter (influxdb||kapacitor||chronograf||telegraf) -> enter the specified container
  logs  (influxdb||kapacitor||chronograf||telegraf) -> stream logs for the specified container

  delete-data  -> delete all data created by the TICK Stack
  docker-clean -> stop and remove all running docker containers
  rebuild-docs -> rebuild the documentation container to see updates
```

To get started just run `./sandbox up`. You browser will open two tabs:

- `localhost:8086` - InfluxDB's address. You will use this as a management UI for the Influx DB and dashboards etc starting from 2.0
- `localhost:8888` - Chronograf's address. In 1.x Chronograf was the central tool. 
                     Below we will be doing some steps to ensure backward compatibility with chronograf. 
- `localhost:3010` - Documentation server. This contains a simple markdown server for tutorials and documentation.


> NOTE: Make sure to stop any existing installations of `influxdb`, `kapacitor` or `chronograf`. If you have them running the Sandbox will run into port conflicts and fail to properly start. In this case stop the existing processes and run `./sandbox restart`. Also make sure you are **not** using _Docker Toolbox_.

Once the Sandbox launches, you should see your dashboard appear in your browser:

### Configuration Steps 
Once sandbox is up, login to the influx db admin page: http://localhost:8086/
Enter these details:
```
Org: org
User: admin 
Password: admin123
Bucket: telegraf (Bucket name has to be telegraf for chornograf to work) 
```
Ensure the database initialization is completed. 

Copy the Admin's token from Data-> API Tokens screen and paste in telegraf.conf and kapacitor.conf. 

Restart telegraf docker from docker desktop and check status. 
Once telegraf is running, you can check the system metrics coming inside the telegraf database. 

Restart kapacitor docker container from docker desktop and check status. 

From Chronograf you can add the database into the config as given here:
https://docs.influxdata.com/chronograf/v1.9/administration/creating-connections/#manage-influxdb-connections-using-the-chronograf-ui

Influx DB config:
```
URL for influxDB: http://influxdb:8086
Org: org
Token: Value from API Tokens screen
Database: telegraf
Retention Policy: autogen 
```
Kapacitor config:
```
URL for Kapacitor: http://kapacitor:9092
Leave other fields blank/default. 
```

Influx DB 2.0 does not have databases and retention policies, it has buckets which combine both. To support tools like Chronograf, we need to create mapping between our bucket and db, rp combination. For this login to the influxdb: 

./sandbox enter influxdb

To map the bucket as per the v1.0 requirement enter below command. Token value is from API Tokens screen, and bucket-id is from Data->Buckets screen. 

```
influx v1 dbrp create \
   --org=org \
   --token=LLDY7DAD3VLT_MytcF1AX7A72qOgEFlR3GG98HNGp0Now0zaz0H1ACWcZoI6xQb5-qiT8LR8jPpTzQGJUEKGlg== \
   --db telegraf \
   --rp autogen \
   --bucket-id ff28b87f09e61d21 --default
```

### Grafana
Grafana can be accessed at localhost:3000 

Default credentials for Grafana: admin/admin 

Enter these and change the default password. 

Configuration:
Left side menu (+ icon)  -> create -> Dashboard -> Create a sample dashboard. Save. 

Left side menu (gear icon) -> configuration - Data Source (Select influxdb) 

Change query language to Flux (influxql only supported in 1.x)

```
 HTTP url:  http://influxdb:8086 
 Database: telegraf
 Org: org
 Token: value from the API Tokens
 ```
 
 
Click Save & Test


Reference: 
https://www.influxdata.com/blog/running-influxdb-2-0-and-telegraf-using-docker/

https://www.sqlpac.com/en/documents/influxdb-v2-getting-started-setup-preparing-migration-from-version-1.7.html

https://docs.influxdata.com/influxdb/v2.0/tools/chronograf/


https://github.com/influxdata/chronograf/issues/2830
=======
![Dashboard](./documentation/static/images/sandbox-dashboard.png)

### InfluxDB
Inserting data into InfluxDB CLI:
Login to the influxDB docker container from docker desktop. Enter the command 
```
influx
```
to start CLI
```
SHOW DATABASES - Lists databases
USE <DATABASE_NAME> - Select this DB for next commands. 
SHOW MEASUREMENTS - lists measurements
```
Example Insert command: 
```
INSERT redis,host=serverA,region=us_west value=0.64
```
This will insert one row into the redis measurement, which if did not exist till then will get created as well. 


### Querying influx DB:

To login to the influxdb cli, open a terminal on the influxdb container from docker desktop or using command prompt:
docker exec -it [<container_name ]  /bin/sh 
```
influx -precision rfc3339 (Precision setting allows to show timestamps in human-readable form)
```

```
show databases -> shows available databases
use [databasename] -> select the database for next commands
show measurements -> shows measurements
fields are numeric,non-indexed and tags are alphanumeric-indexed (mostly). Tags are strings.
show field keys from cpu - see all field names from cpu measurements
show tag keys from cpu - see all tag names from cpu measurements 
show tag values from cpu - see all tag values from cpu measurements
SHOW TAG VALUES CARDINALITY FROM cpu WITH KEY = "cpu" - number of distinct values of the tag 'cpu' in the measurement 'cpu'
```

At least one field column should be given in SELECT statement. 
We cannot use OR on the time values. 
Strings should be enclosed in quotes. 
In some error cases, no error is returned, and no data also. 
Cannot use maths within functions, ie not possible to do avg(gross-net)

GROUP BY TIME 
```
SELECT mean("water_level") FROM "h2o_feet" where time>='2019-09-17T20:00:00Z' group by time(10m)
SELECT mean("water_level") FROM "h2o_feet" where time>='2019-09-17T20:00:00Z' group by time(10m),location
SELECT mean("water_level") FROM "h2o_feet" where time>='2019-09-17T20:00:00Z' group by location, time(10m) fill(0)
```

CTAS in influxdb
```
SELECT * INTO "NOAA_water_database"."autogen".water_levels_bkp FROM h2o_feet GROUP BY *
```
ORDER BY TIME
ORDER by time DESC

```
SELECT mean("water_level") FROM "h2o_feet" WHERE "location" = 'santa_monica' AND time>='2019-09-17T20:00:00Z'  GROUP BY TIME(5m) ORDER BY time DESC LIMIT 5
```

Pagination
```
 SELECT "water_level","location" FROM "h2o_feet" LIMIT 3 OFFSET 3

 SELECT mean("water_level") FROM "h2o_feet" where time>='2019-09-17T20:00:00Z' tz('America/Chicago')
 SELECT mean("water_level") FROM "h2o_feet" where time>='2019-09-17T20:00:00Z' tz('Asia/Calcutta')

 SELECT "water_level" FROM "h2o_feet" WHERE time > now() - 1h

 SELECT "level description" FROM "h2o_feet" WHERE time >= '2019-09-17T21:36:00Z'
```

### InfluxDB Relay 
Influxdb relay adds a high availability layer to the InfluxDB. Here for demo purpose, there is only one backend on the Relay, not real high availability :) 
To write to the InfluxDB using the relay endpoint, below command can be used from the Relay container terminal: 

```
curl -i -XPOST 'http://localhost:9096/write?db=telegraf' \
--data-binary 'relay_test,host=server01,region=us-west value=0.96 1647873106000000000'
```

This will internally call the backend InfluxDB url using the command as below: 
```
curl -i -XPOST 'http://influxdb:8086/write?db=telegraf' \
--data-binary 'relay_test,host=server01,region=us-west value=0.96 1647873106000000000'
```

Data can be verified on the influxdb container terminal like so:
```
 select * from relay_test
```
Relay can be pinged by the below command. Look for X-Influxdb-Version: relay in the output
```
sh-4.4# curl -kv http://localhost:9096/ping
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 9096 (#0)
> GET /ping HTTP/1.1
> Host: localhost:9096
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 204 No Content
< X-Influxdb-Version: relay
< Date: Mon, 21 Mar 2022 16:01:37 GMT
< 
* Connection #0 to host localhost left intact
```



