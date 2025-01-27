# Module 1 Homework: Docker & SQL

## Question 1. Understanding docker first run 

Run docker with the `python:3.12.8` image in an interactive mode, use the entrypoint `bash`.

What's the version of `pip` in the image?

- 24.3.1
- 24.2.1
- 23.3.1
- 23.2.1

## Answer: 
> 24.3.1

### Proof of Work:
```bash
oscos@dedt02:~/de_zoomcamp/01-docker-terraform/homework_q1$ docker run -it --entrypoint=bash python:3.12.8
Unable to find image 'python:3.12.8' locally
3.12.8: Pulling from library/python
fd0410a2d1ae: Pull complete 
bf571be90f05: Pull complete 
684a51896c82: Pull complete 
fbf93b646d6b: Pull complete 
5f16749b32ba: Pull complete 
e00350058e07: Pull complete 
eb52a57aa542: Pull complete 
Digest: sha256:5893362478144406ee0771bd9c38081a185077fb317ba71d01b7567678a89708
Status: Downloaded newer image for python:3.12.8
root@1ebd24d1d615:/# pip list
Package Version
------- -------
pip     24.3.1
```

## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that **pgadmin** should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

- postgres:5433
- localhost:5432
- db:5433
- postgres:5432
- db:5432

If there are more than one answers, select only one of them

## Answer: 
> db:5432

### Host Name Explanation:

While running both the service name `db` and the container_name `postgres` returned the same results, I believe the service name is preferred.

### Proof of Work:

Using `ping` to check for hostname and network connectivity:

```bash
oscos@dedt02:~/de_zoomcamp/01-docker-terraform/homework_q2$ docker exec -it pgadmin /bin/sh
/pgadmin4 $ ping db
PING db (172.20.0.2): 56 data bytes
64 bytes from 172.20.0.2: seq=0 ttl=42 time=0.069 ms
64 bytes from 172.20.0.2: seq=1 ttl=42 time=0.058 ms
64 bytes from 172.20.0.2: seq=2 ttl=42 time=0.079 ms
^C
--- db ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.058/0.068/0.079 ms
/pgadmin4 $ ping postgres
PING postgres (172.20.0.2): 56 data bytes
64 bytes from 172.20.0.2: seq=0 ttl=42 time=0.060 ms
64 bytes from 172.20.0.2: seq=1 ttl=42 time=0.070 ms
64 bytes from 172.20.0.2: seq=2 ttl=42 time=0.172 ms
^C
--- postgres ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.060/0.100/0.172 ms
/pgadmin4 $ 
```

Use `getent hosts` to resolve the hostname to its ip address:

```console
oscos@dedt02:~/de_zoomcamp/01-docker-terraform/homework_q2$ docker exec -it pgadmin /bin/sh
/pgadmin4 $ getent hosts db
172.20.0.2        db  db
/pgadmin4 $ getent hosts postgres
172.20.0.2        postgres  postgres
/pgadmin4 $ 
```

### Port Number Explanation:

Within the docker compose network, the internal port is used.

### Proof of Work:

```
/pgadmin4 $ nc -zv db 5432
db (172.20.0.2:5432) open
/pgadmin4 $ nc -zv postgres 5432
postgres (172.20.0.2:5432) open
/pgadmin4 $ 
```

##  Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from October 2019:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
```

You will also need the dataset with zones:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

Download this data and put it into Postgres.

You can use the code from the course. It's up to you whether
you want to use Jupyter or a python script.

## Question 3. Trip Segmentation Count

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, **respectively**, happened:
1. Up to 1 mile
2. In between 1 (exclusive) and 3 miles (inclusive),
3. In between 3 (exclusive) and 7 miles (inclusive),
4. In between 7 (exclusive) and 10 miles (inclusive),
5. Over 10 miles 

Answers:

- 104,802;  197,670;  110,612;  27,831;  35,281
- 104,802;  198,924;  109,603;  27,678;  35,189
- 104,793;  201,407;  110,612;  27,831;  35,281
- 104,793;  202,661;  109,603;  27,678;  35,189
- 104,838;  199,013;  109,645;  27,688;  35,202

## Answer: 
> 104,802;  198,924;  109,603;  27,678;  35,189

### Proof of Work: 

### Q3.1
```SQL
SELECT COUNT(*)
FROM green_taxi_trips AS g
WHERE 
	g.lpep_pickup_datetime >= '2019-10-01 00:00:00' 
	AND 
	lpep_dropoff_datetime < '2019-11-01 00:00:00'
	AND 
	trip_distance <= 1
LIMIT 1;

-- Query Result: 104,802
```

## Q3.2
```sql
SELECT COUNT(*)
FROM green_taxi_trips AS g
WHERE 
	g.lpep_pickup_datetime >= '2019-10-01 00:00:00'
	AND
	lpep_dropoff_datetime < '2019-11-01 00:00:00'
	AND 
	trip_distance > 1 
	AND 
	trip_distance <= 3
LIMIT 1;

-- Query Result: 198,924 
```

## Q3.3
```sql
SELECT COUNT(*) 
FROM green_taxi_trips AS g
WHERE 
	g.lpep_pickup_datetime >= '2019-10-01 00:00:00' 
	AND
	lpep_dropoff_datetime < '2019-11-01 00:00:00'
	AND 
	trip_distance > 3 
	AND 
	trip_distance <= 7
LIMIT 1;

-- Query Result: 109,603
```

## Q3.4 
```sql
SELECT COUNT(*) 
FROM green_taxi_trips AS g
WHERE 
	g.lpep_pickup_datetime >= '2019-10-01 00:00:00' 
	AND
	lpep_dropoff_datetime < '2019-11-01 00:00:00'
	AND 
	trip_distance > 7 
	AND 
	trip_distance <= 10
LIMIT 1;

-- Query Result: 27,678
```


## Q3.5
```sql
SELECT COUNT(*) 
FROM green_taxi_trips AS g
WHERE 
	g.lpep_pickup_datetime >= '2019-10-01 00:00:00' 
	AND 
	lpep_dropoff_datetime < '2019-11-01 00:00:00'
	AND 
	trip_distance > 10
LIMIT 1;

-- Query Result: 35,189
```

## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance?
Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance. 

- 2019-10-11
- 2019-10-24
- 2019-10-26
- 2019-10-31

## Answer: 
> 2019-10-31

### Proof of Work

## Q4
```SQL
SELECT g.lpep_pickup_datetime, trip_distance  
FROM green_taxi_trips AS g
ORDER BY trip_distance DESC
LIMIT 1;
```
Query Results:

| lpep_pickup_datetime | trip_distance |
|----------------------|---------------|
| 2019-10-31 23:23:41  | 515.89        |


## Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in
`total_amount` (across all trips) for 2019-10-18?

Consider only `lpep_pickup_datetime` when filtering by date.
 
- East Harlem North, East Harlem South, Morningside Heights
- East Harlem North, Morningside Heights
- Morningside Heights, Astoria Park, East Harlem South
- Bedford, East Harlem North, Astoria Park

## Answer: 
> East Harlem North, East Harlem South, Morningside Heights

### Proof of Work:

## Q5
```sql
SELECT q.* FROM 
(
	SELECT z."Zone", SUM(g.total_amount) AS sum_total_amount
	FROM green_taxi_trips AS g
	INNER JOIN taxi_zone_lookup AS z ON g."PULocationID" = z."LocationID"
	WHERE 
		g.lpep_pickup_datetime >= '2019-10-18 00:00:00' and g.lpep_pickup_datetime < '2019-10-19 00:00:00'
	GROUP BY z."Zone"
) AS q
ORDER BY q.sum_total_amount DESC
LIMIT 3;
```
Query Resuls:

| Zone                 | sum_total_amount        |
|----------------------|--------------------------|
| East Harlem North    | 18686.680000000022       |
| East Harlem South    | 16797.26000000007        |
| Morningside Heights  | 13029.790000000045       |


## Question 6. Largest tip

For the passengers picked up in October 2019 in the zone
named "East Harlem North" which was the drop off zone that had
the largest tip?

Note: it's `tip` , not `trip`

We need the name of the zone, not the ID.

- Yorkville West
- JFK Airport
- East Harlem North
- East Harlem South

## Answer: 
> JFK Airport

#### Proof of Work
```sql
SELECT 
	p."Zone" AS "pick_up_zone", 
	d."Zone" AS "drop_off_zone", 
	g.tip_amount
FROM green_taxi_trips AS g
INNER JOIN taxi_zone_lookup AS p 
	ON g."PULocationID" = p."LocationID"
INNER JOIN taxi_zone_lookup AS d 
	ON g."DOLocationID" = d."LocationID"
WHERE 
	g.lpep_pickup_datetime >= '2019-10-01 00:00:00'
	AND 
	g.lpep_pickup_datetime < '2019-11-01 00:00:00'
	AND
	p."Zone" = 'East Harlem North'
ORDER BY g.tip_amount DESC
LIMIT 1
```
Query Results:

| pick_up_zone      | drop_off_zone | tip_amount |
|-------------------|---------------|------------|
| East Harlem North | JFK Airport   | 87.3       |

## Terraform

In this section homework we'll prepare the environment by creating resources in GCP with Terraform.

In your VM on GCP/Laptop/GitHub Codespace install Terraform. 
Copy the files from the course repo
[here](../../../01-docker-terraform/1_terraform_gcp/terraform) to your VM/Laptop/GitHub Codespace.

Modify the files as necessary to create a GCP Bucket and Big Query Dataset.

## Question 7. Terraform Workflow

Which of the following sequences, **respectively**, describes the workflow for: 
1. Downloading the provider plugins and setting up backend,
2. Generating proposed changes and auto-executing the plan
3. Remove all resources managed by terraform`

Answers:
- terraform import, terraform apply -y, terraform destroy
- teraform init, terraform plan -auto-apply, terraform rm
- terraform init, terraform run -auto-approve, terraform destroy
- terraform init, terraform apply -auto-approve, terraform destroy
- terraform import, terraform apply -y, terraform rm

## Answer: 
> terraform init, terraform apply -auto-approve, terraform destroy

### Proof of Work

```bash
oscos@dedt02:~/de_zoomcamp/01-docker-terraform/1_terraform$ terraform init
Initializing the backend...
Initializing provider plugins...
- Finding hashicorp/google versions matching "6.16.0"...
- Installing hashicorp/google v6.16.0...
- Installed hashicorp/google v6.16.0 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
oscos@dedt02:~/de_zoomcamp/01-docker-terraform/1_terraform$ terraform apply -auto-approve

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_storage_bucket.demo-bucket-01 will be created
  + resource "google_storage_bucket" "demo-bucket-01" {
      + effective_labels            = {
          + "goog-terraform-provisioned" = "true"
        }
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "US"
      + name                        = "dezc2025-terra-demo-bucket-01"
      + project                     = (known after apply)
      + project_number              = (known after apply)
      + public_access_prevention    = (known after apply)
      + rpo                         = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + terraform_labels            = {
          + "goog-terraform-provisioned" = "true"
        }
      + uniform_bucket_level_access = (known after apply)
      + url                         = (known after apply)

      + lifecycle_rule {
          + action {
              + type          = "Delete"
                # (1 unchanged attribute hidden)
            }
          + condition {
              + age                    = 3
              + matches_prefix         = []
              + matches_storage_class  = []
              + matches_suffix         = []
              + with_state             = (known after apply)
                # (3 unchanged attributes hidden)
            }
        }
      + lifecycle_rule {
          + action {
              + type          = "AbortIncompleteMultipartUpload"
                # (1 unchanged attribute hidden)
            }
          + condition {
              + age                    = 1
              + matches_prefix         = []
              + matches_storage_class  = []
              + matches_suffix         = []
              + with_state             = (known after apply)
                # (3 unchanged attributes hidden)
            }
        }

      + soft_delete_policy (known after apply)

      + versioning (known after apply)

      + website (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
google_storage_bucket.demo-bucket-01: Creating...
google_storage_bucket.demo-bucket-01: Creation complete after 2s [id=dezc2025-terra-demo-bucket-01]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

oscos@dedt02:~/de_zoomcamp/01-docker-terraform/1_terraform$ terraform destroy
google_storage_bucket.demo-bucket-01: Refreshing state... [id=dezc2025-terra-demo-bucket-01]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_storage_bucket.demo-bucket-01 will be destroyed
  - resource "google_storage_bucket" "demo-bucket-01" {
      - default_event_based_hold    = false -> null
      - effective_labels            = {
          - "goog-terraform-provisioned" = "true"
        } -> null
      - enable_object_retention     = false -> null
      - force_destroy               = true -> null
      - id                          = "dezc2025-terra-demo-bucket-01" -> null
      - labels                      = {} -> null
      - location                    = "US" -> null
      - name                        = "dezc2025-terra-demo-bucket-01" -> null
      - project                     = "dezc2025" -> null
      - project_number              = 575796905162 -> null
      - public_access_prevention    = "inherited" -> null
      - requester_pays              = false -> null
      - rpo                         = "DEFAULT" -> null
      - self_link                   = "https://www.googleapis.com/storage/v1/b/dezc2025-terra-demo-bucket-01" -> null
      - storage_class               = "STANDARD" -> null
      - terraform_labels            = {
          - "goog-terraform-provisioned" = "true"
        } -> null
      - uniform_bucket_level_access = false -> null
      - url                         = "gs://dezc2025-terra-demo-bucket-01" -> null

      - hierarchical_namespace {
          - enabled = false -> null
        }

      - lifecycle_rule {
          - action {
              - type          = "Delete" -> null
                # (1 unchanged attribute hidden)
            }
          - condition {
              - age                                     = 3 -> null
              - days_since_custom_time                  = 0 -> null
              - days_since_noncurrent_time              = 0 -> null
              - matches_prefix                          = [] -> null
              - matches_storage_class                   = [] -> null
              - matches_suffix                          = [] -> null
              - num_newer_versions                      = 0 -> null
              - send_age_if_zero                        = false -> null
              - send_days_since_custom_time_if_zero     = false -> null
              - send_days_since_noncurrent_time_if_zero = false -> null
              - send_num_newer_versions_if_zero         = false -> null
              - with_state                              = "ANY" -> null
                # (3 unchanged attributes hidden)
            }
        }
      - lifecycle_rule {
          - action {
              - type          = "AbortIncompleteMultipartUpload" -> null
                # (1 unchanged attribute hidden)
            }
          - condition {
              - age                                     = 1 -> null
              - days_since_custom_time                  = 0 -> null
              - days_since_noncurrent_time              = 0 -> null
              - matches_prefix                          = [] -> null
              - matches_storage_class                   = [] -> null
              - matches_suffix                          = [] -> null
              - num_newer_versions                      = 0 -> null
              - send_age_if_zero                        = false -> null
              - send_days_since_custom_time_if_zero     = false -> null
              - send_days_since_noncurrent_time_if_zero = false -> null
              - send_num_newer_versions_if_zero         = false -> null
              - with_state                              = "ANY" -> null
                # (3 unchanged attributes hidden)
            }
        }

      - soft_delete_policy {
          - effective_time             = "2025-01-26T21:35:37.284Z" -> null
          - retention_duration_seconds = 604800 -> null
        }
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

google_storage_bucket.demo-bucket-01: Destroying... [id=dezc2025-terra-demo-bucket-01]
google_storage_bucket.demo-bucket-01: Destruction complete after 1s

Destroy complete! Resources: 1 destroyed.
oscos@dedt02:~/de_zoomcamp/01-docker-terraform/1_terraform$ 

```

## Submitting the solutions

* Form for submitting: https://courses.datatalks.club/de-zoomcamp-2025/homework/hw1
