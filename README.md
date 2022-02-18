# Custom TICK stack Sandbox
This is a custom TICK stack sandbox with:  
TELEGRAF 1.13
INFLUXDB 1.8
CHRONOGRAF latest
KAPACITOR 1.5.5
GRAFANA 7.1.0 

### Running

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

- `localhost:8888` - Chronograf's address. You will use this as a management UI for the full stack
- `localhost:3010` - Documentation server. This contains a simple markdown server for tutorials and documentation.

### Grafana
Grafana can be accessed at localhost:3000 
Default credentials for Grafana: admin/admin 
Enter these and change the default password. 

Configuration:
Left side menu (+ icon)  -> create -> Dashboard -> Create a sample dashboard. Save. 
Left side menu (gear icon) -> configuration - Data Source 
 HTTP url:  http://influxdb:8086 
 Database: telegraf
Click Save & Test

> NOTE: Make sure to stop any existing installations of `influxdb`, `kapacitor` or `chronograf`. If you have them running the Sandbox will run into port conflicts and fail to properly start. In this case stop the existing processes and run `./sandbox restart`. Also make sure you are **not** using _Docker Toolbox_.

Once the Sandbox launches, you should see your dashboard appear in your browser:

![Dashboard](./documentation/static/images/landing-page.png)

You are ready to get started with the TICK Stack!

Click the Host icon in the left navigation bar to see your host (named `telegraf-getting-started`) and its overall status.
![Host List](./documentation/static/images/host-list.png)

You can click on `system` hyperlink to see a pre-built dashboard visualizing the basic system stats for your
host, then check out the tutorials at `http://localhost:3010/tutorials`.

If you are using the nightly builds and want to get started with Flux, make sure you check out the [Getting Started with Flux](./documentation/static/tutorials/flux-getting-started.md) tutorial.

> Note: see [influx-stress](https://github.com/influxdata/influx-stress) to create data for your Sandbox.

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


