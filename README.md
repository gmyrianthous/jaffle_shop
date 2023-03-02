# Containerised Jaffle Shop dbt Project
This is a containerised version of the popular 
[Jaffle Shop dbt project published by dbt Labs](https://github.com/dbt-labs/jaffle_shop). 
You can use this project to populate and run Jaffle Shop in a Postgres instance.

# Running the project with Docker
You'll first need to build and run the services via Docker (as defined in `docker-compose.yml`):
```bash
$ docker-compose build
$ docker-compose up
```

The commands above will run a Postgres instance and then build the dbt resources of Jaffle Shop as specified in the
repository. These containers will remain up and running so that you can:
- Query the Postgres database and the tables created out of dbt models
- Run further dbt commands via dbt CLI


## Building additional or modified data models
Once the containers are up and running, you can still make any modifications in the existing dbt project 
and re-run any command to serve the purpose of the modifications. 

In order to build your data models, you first need to access the container.

To do so, we infer the container id for `dbt` running container:
```bash
docker ps
```

Then enter the running container:
```bash
docker exec -it <container-id> /bin/bash
```

And finally:

```bash
# Install dbt deps (might not required as long as you have no -or empty- `dbt_packages.yml` file)
dbt deps

# Build seeds
dbt seeds --profiles-dir profiles

# Build data models
dbt run --profiles-dir profiles

# Build snapshots
dbt snapshot --profiles-dir profiles

# Run tests
dbt test --profiles-dir profiles
```

Alternatively, you can run everything in just a single command:

```bash
dbt build --profiles-dir profiles
```

## Querying Jaffle Shop data models on Postgres
In order to query and verify the seeds, models and snapshots created in the dummy dbt project, simply follow the 
steps below. 

Find the container id of the postgres service:
```commandline
docker ps 
```

Then run 
```commandline
docker exec -it <container-id> /bin/bash
```

We will then use `psql`, a terminal-based interface for PostgreSQL that allows us to query the database:
```commandline
psql -U postgres
```

You can list tables and views as shown below:
```bash
postgres=# \dt
             List of relations
 Schema |     Name      | Type  |  Owner   
--------+---------------+-------+----------
 public | customers     | table | postgres
 public | orders        | table | postgres
 public | raw_customers | table | postgres
 public | raw_orders    | table | postgres
 public | raw_payments  | table | postgres
(5 rows)

postgres=# \dv
            List of relations
 Schema |     Name      | Type |  Owner   
--------+---------------+------+----------
 public | stg_customers | view | postgres
 public | stg_orders    | view | postgres
 public | stg_payments  | view | postgres
(3 rows)

```

Now you can query the tables constructed form the seeds, models and snapshots defined in the dbt project:
```sql
SELEC * FROM <table_or_view_name>;
```
