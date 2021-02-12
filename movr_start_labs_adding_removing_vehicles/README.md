# MovR

This repo contains the source code for an example implementation of a single-region application for the fictional vehicle-sharing company MovR, as used by the CockroachDB for Python Developers course.

- [Overview](#overview) gives a high-level overview the MovR application stack. 

## Overview

The application stack consists of the following components:

- A CockroachCloud instance
- Python class definitions that map to the table(s) in the database.
- A backend API that defines the application's connection to the database and the database transactions.
- A Flask server that handles requests from client web browsers.
- HTML templates that define web pages that the Flask server renders.

## new table
* break original `vehicles` table into 1:many relationship child table: `location_history`.
    1. update database:
    ```
    # Create location_history table
    CREATE TABLE location_history (
        id UUID PRIMARY KEY,
        vehicle_id UUID REFERENCES vehicles(id) ON DELETE CASCADE,  # FK, and parent row delete policy
        ts TIMESTAMP NOT NULL,
        longitude FLOAT8 NOT NULL,
        latitude FLOAT8 NOT NULL
    );

    # Migrate the data
    INSERT INTO movr.location_history (id, vehicle_id, ts, longitude, latitude)
    SELECT gen_random_uuid(), id, last_checkin, last_longitude, last_latitude
    FROM vehicles;

    # Remove the columns from parent table
    SET sql_safe_updates = false;
    ALTER TABLE vehicles DROP COLUMN last_checkin,
        DROP COLUMN last_longitude,
        DROP COLUMN last_latitude;
    ```

## Setup

### Use single-node lcoal cockroachdb
```
# fxrc @ popos in ~/Tools [18:55:16] C:130
$ cockroach start-single-node --insecure
...
CockroachDB node starting at 2021-02-11 02:55:42.459646733 +0000 UTC (took 0.1s)
build:               CCL v20.2.4 @ 2021/01/21 00:08:24 (go1.13.14)
webui:               http://popos:8080
sql:                 postgresql://root@popos:26257?sslmode=disable
RPC client flags:    cockroach <client cmd> --host=popos:26257 --insecure
logs:                /home/fxrc/Tools/cockroach-data/logs
temp dir:            /home/fxrc/Tools/cockroach-data/cockroach-temp687873191
external I/O path:   /home/fxrc/Tools/cockroach-data/extern
store[0]:            path=/home/fxrc/Tools/cockroach-data
storage engine:      pebble
status:              restarted pre-existing node
clusterID:           f50518bb-b723-4859-85f1-c70762955ee7
nodeID:              1
```

### Tried CockroachCloud (beta)
* failed to import csv:
```
tony@free-tier3.aws-us-west-2.cockroachlabs.cloud:26257/movr> IMPORT TABLE movr.vehicles (                                                            
        id UUID PRIMARY KEY,                                                                                                                          
        last_longitude FLOAT8,                                                                                                                        
        last_latitude FLOAT8,                                                                                                                         
        battery INT8,                                                                                                                                 
        last_checkin TIMESTAMP,                                                                                                                       
        in_use BOOL,                                                                                                                                  
        vehicle_type STRING NOT NULL                                                                                                                  
    )                                                                                                                                                 
CSV DATA ('https://cockroach-university-public.s3.amazonaws.com/10000vehicles.csv')                                                                   
    WITH delimiter = '|';                                                                                                                             
ERROR: creating importTables: restoring table desc and namespace entries: failed to lookup parent DB 52: RangeIterator failed to seek to /Table/3/1/52
/2/1: rpc error: code = Unauthenticated desc = requested key /Table/3/1/52/2/1 not fully contained in tenant keyspace /Tenant/7{5-6}
```

### Database and Environment Setup

You'll need to connect to a [CockroachCloud](https://cockroachlabs.cloud/) cluster.

1. Configure environment variables
     
    When running your server, it's easiest if your CockroachCloud connection
    string is assigned to the DB_URI variable in the `.env` file. You can paste
    it in with the editor of your choice.

2. Initialize the database

    You can do this with

    ~~~ shell
    $ cat dbinit.sql | cockroach sql --url <cockroachcloud_url>
    ~~~
    
### Application setup

1. To run the application, use the following command in the shell:

    ~~~ shell
    $ ./server.py run
    ~~~

1. Navigate to the url provided (defaults to [http://localhost:36257](http://localhost:36257)) to use the application.

### Clean up

1. To shut down the application, `Ctrl+C` out of the Python process.

## Note: Testing

We are still building out the testing suite. Please use at your own risk.
