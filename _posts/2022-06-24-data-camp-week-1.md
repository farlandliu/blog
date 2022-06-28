---
toc: true
layout: post
description: # data engineering zoomcamp  week 1 note and practise
categories: [data]
title: data engineering zoomcamp week 1 practise
---

# data engineering zoomcamp week 1 **practise**

## concepts and technology

- Data Lake & Data Warehouse
- Terraform Iac
- Docker & Docker-compose
- SQL: Data Analysis & Exploration
- Airflow: Pipline Orchestration
- DBT: data Transformation
- parquet



## Practise

### 1 setup local environment
ref this [guide](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/main/week_1_basics_n_setup)

1). install docker and docker-compose
2). install pgcli

```shell

sudo apt-get install libpq-dev python3-dev python3-venv
mkdir zoomcamp
cd zoomcamp & python3 -m venv .venv
source .venv/bin/activate
pip install pip -U
pip install pgcli

```

### 2. run Postgres in docker 

![run postgres in docker]({{ site.baseurl }}/images/pgstart.png)

```shell
$ bash pg.sh
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.utf8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgresql/data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Etc/UTC
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    pg_ctl -D /var/lib/postgresql/data -l logfile start

waiting for server to start....2022-06-27 07:38:35.587 UTC [49] LOG:  starting PostgreSQL 13.5 (Debian 13.5-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
2022-06-27 07:38:35.587 UTC [49] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2022-06-27 07:38:35.590 UTC [50] LOG:  database system was shut down at 2022-06-27 07:38:35 UTC
2022-06-27 07:38:35.595 UTC [49] LOG:  database system is ready to accept connections
 done
server started
CREATE DATABASE


/usr/local/bin/docker-entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*

2022-06-27 07:38:35.813 UTC [49] LOG:  received fast shutdown request
waiting for server to shut down....2022-06-27 07:38:35.813 UTC [49] LOG:  aborting any active transactions
2022-06-27 07:38:35.815 UTC [49] LOG:  background worker "logical replication launcher" (PID 56) exited with exit code 1
2022-06-27 07:38:35.816 UTC [51] LOG:  shutting down
2022-06-27 07:38:35.828 UTC [49] LOG:  database system is shut down
 done
server stopped

PostgreSQL init process complete; ready for start up.

2022-06-27 07:38:35.941 UTC [1] LOG:  starting PostgreSQL 13.5 (Debian 13.5-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
2022-06-27 07:38:35.941 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2022-06-27 07:38:35.941 UTC [1] LOG:  listening on IPv6 address "::", port 5432
2022-06-27 07:38:35.942 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2022-06-27 07:38:35.944 UTC [63] LOG:  database system was shut down at 2022-06-27 07:38:35 UTC
2022-06-27 07:38:35.949 UTC [1] LOG:  database system is ready to accept connections

```

connect to postgres using pgcli
```shell
pgcli -h localhost -p 5432 -u root -d ny_taxi
```

![login postgres using pgcli]({{ site.baseurl }}/images/ps-start-2.png)


### 3. import taxi data to Postgres


install the python packages:
```shell
pip install pandas sqlalchemy
```
download the nyc taxi data
```
$ wget https://s3.amazonaws.com/nyc-tlc/csv_backup/yellow_tripdata_2021-01.csv
....
$ wc -l yellow_tripdata_2021-01.csv
1369770 yellow_tripdata_2021-01.csv
```

week1/import_data.py

```python
import pandas as pd
from sqlalchemy import create_engine

# create engine and set the root as postgresql://user:password@host:port/database
engine = create_engine('postgresql://root:root@localhost:5432/ny_taxi')

# build a text iterrator with chunk size 100000
df_iter = pd.read_csv('yellow_tripdata_2021-01.csv', iterator=True, chunksize=100000)

while True:
    df = next(df_iter)
    df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
    df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)
    df.to_sql(name='yellow_taxi_data', con=engine, if_exists='append')

```

run the scripts, but erros occured:

```shell
$ python import_data.py
inserted another chunk..., took 8.551 seconds
inserted another chunk..., took 8.318 seconds
inserted another chunk..., took 8.301 seconds
inserted another chunk..., took 8.303 seconds
inserted another chunk..., took 8.312 seconds
inserted another chunk..., took 8.294 seconds
inserted another chunk..., took 8.454 seconds
inserted another chunk..., took 8.303 seconds
inserted another chunk..., took 8.250 seconds
inserted another chunk..., took 8.291 seconds
inserted another chunk..., took 8.383 seconds
inserted another chunk..., took 8.283 seconds
...

```

set `low_memory=False` in `read_csv` and run again:

```shell
logging

```

login into database and see the nyc taxi data schema:

```shell
$ pgcli -h localhost -p 5432 -u root -d ny_taxi

$ root@localhost:ny_taxi> \d yellow_taxi_data
+-----------------------+-----------------------------+-----------+
| Column                | Type                        | Modifiers |
|-----------------------+-----------------------------+-----------|
| index                 | bigint                      |           |
| VendorID              | bigint                      |           |
| tpep_pickup_datetime  | timestamp without time zone |           |
| tpep_dropoff_datetime | timestamp without time zone |           |
| passenger_count       | bigint                      |           |
| trip_distance         | double precision            |           |
| RatecodeID            | bigint                      |           |
| store_and_fwd_flag    | text                        |           |
| PULocationID          | bigint                      |           |
| DOLocationID          | bigint                      |           |
| payment_type          | bigint                      |           |
| fare_amount           | double precision            |           |
| extra                 | double precision            |           |
| mta_tax               | double precision            |           |
| tip_amount            | double precision            |           |
| tolls_amount          | double precision            |           |
| improvement_surcharge | double precision            |           |
| total_amount          | double precision            |           |
| congestion_surcharge  | double precision            |           |
| airport_fee           | double precision            |           |
+-----------------------+-----------------------------+-----------+
Indexes:
    "ix_yellow_taxi_data_index" btree (index)

Time: 0.083s

> SELECT COUNT(1) FROM yellow_taxi_data;
+---------+
| count   |
|---------|
| 6848845 |
+---------+
SELECT 1
Time: 1.690s (1 second), executed in: 1.675s (1 second)
```

### 4. docker-compose 

got error:
```shell
$ docker network create pg-network

$ docker-compose up -d
ERROR: The Compose file './docker-compose.yaml' is invalid because:
Unsupported config option for services: 'pgadmin'

$ docker-compose -v
docker-compose version 1.25.0, build unknown
```

add `version: '3.3'` to docker-compose.yaml and run again

```shell
$ docker-compose up -d
Creating network "week1_default" with the default driver
Creating week1_pgdatabase_1 ... done
Creating week1_pgadmin_1    ... done

```

### 5. build a simple pipline with docker

```shell
docker build -t taxi_ingest:v001 .
```
Then run the docker.
```shell
URL="http://172.24.208.1:8000/yellow_tripdata_2021-01.csv"

$ docker run -it \
  --network=week1_default \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pg-database \
    --db=ny_taxi \
    --port=5432 \
    --table_name=yellow_taxi_trips \
    --url=${URL}
```
Got an error:
```shell
sqlalchemy.exc.OperationalError: (psycopg2.OperationalError) could not translate host name "pg-database" to address: Temporary failure in name resolution

# need change the postgres host per docke-compose.yaml
# use hostname: pgdatabase
$ docker run -it \
  --network=week1_default \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pgdatabase \
    --db=ny_taxi \
    --port=5432 \
    --table_name=yellow_taxi_trips \
    --url=${URL}
inserted another chunk..., took 9.058 seconds
inserted another chunk..., took 9.044 seconds
inserted another chunk..., took 8.881 seconds
inserted another chunk..., took 8.875 seconds
inserted another chunk..., took 8.963 seconds
```
Then access pgAdmin by `localhost:8080`, username: `admin@admin.com`, pass:`root`
![pgAdmin4]({{ site.baseurl }}/images/pgadmin4-1.png).

Add a host: pgdatabase, user:root, pass:root.
View the table `yellow_taxi_data` and make a query.

![table schema]({{ site.baseurl }}/images/pgadmin4-2.png)

![query nyx taxi records]({{ site.baseurl }}/images/pgadmin4-3.png)


## Additional Note

When import parquet format into database, there was an error:
```shell
import_data.py:13: DtypeWarning: Columns (6) have mixed types. Specify dtype option on import or set low_memory=False.
  for chunk in df_iter:
inserted another chunk..., took 8.111 seconds
inserted another chunk..., took 5.250 seconds
```

The records  in the table  only has 6848845 rows.
**TODO** Fix error in parquet format.

### how to fix column(6) mixed data type?

col(6)='store_and_fwd_flag' is char of 'Y' OR 'N',

```
1,2021-01-01 00:30:10.000000,2021-01-01 00:36:12.000000,1,2.1,1,"N",142,43,2,8,3,0.5,0,0,0.3,11.8,2.5,

```