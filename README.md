```bash
ssh -p 99999 watsonx@geo.services.cloud.techzone.ibm.com
```
password: watsonx.data

```bash
sudo su -
```


4.5.1   Stopping watsonx.data
```bash
sudo su -
cd /root/ibm-lh-dev/bin
./stop
./status --all
```

4.5.2. Starting watsonx.data
```bash
export LH_RUN_MODE=diag
./start
./status --all
```

5.1.6
**Username:** ibmlhadmin  
**Password:** password

5.3.11
[Download cars.csv](https://github.com/aseelert/watsonx-data-enablement-l3/raw/main/cars.csv)


5.4.2
```sql  
select car, avg(mpg) as avg_mpg from iceberg_data.my_schema.cars
group by car order by car;
```

5.6.2
```bash
/root/ibm-lh-dev/bin/user-mgmt add-user user1 User password1
```

6.1.2
```bash
cd /root/ibm-lh-dev/bin
./presto-cli
```
```bash
show catalogs;
```
```bash
select count(*) from tpch.tiny.customer;
```
```bash
show schemas in tpch;
```
```bash
use tpch.tiny;
```
```bash
select count(*) from customer;
```
```bash
quit;
```
```bash
./presto-cli --catalog tpch --schema tiny
```
```bash
select count(*) from customer;
```
```bash
quit;
```
```bash
./presto-cli --catalog iceberg_data 
```
```bash
create schema if not exists newschema with (location='s3a://iceberg-bucket/newschema');
```
```bash
show schemas;
```
```bash
create table newschema.users (id int, name varchar, age int);
```
```bash
insert into newschema.users values (1, 'Robert', 54);
insert into newschema.users values (2, 'Susan', 37);
select * from newschema.users order by id;
```
```bash
use newschema;
show tables;
quit;
```

7.2. Exploring MinIO Object Storage
```bash
docker exec ibm-lh-presto printenv | grep LH_S3_ACCESS_KEY | sed's/.*=//'
```
```bash
docker exec ibm-lh-presto printenv | grep LH_S3_SECRET_KEY | sed 's/.*=//'
```

8.8

You can download the `aircraft.parquet` file from the following URL:

[Download aircraft.parquet](https://github.com/aseelert/watsonx-data-enablement-l3/raw/main/aircraft.parquet)

```bash
/root/ibm-lh-dev/bin/presto-cli
```
```bash
create schema hive_data.myschema2 with (location = 's3a://hive-bucket/myschema2');
```
```sql
create table hive_data.myschema2.aircraft
(tail_number varchar,
manufacturer varchar,
model varchar) 
with (format = 'Parquet',
external_location='s3a://hive-bucket/myschema2/aircraft');
```
```sql
select count(*) from hive_data.myschema2.aircraft;
quit;
```

9.4 Federated Queries

- **Display name**: `Db2DW`
- **Database name**: `GOSALES`
- **Hostname**: `192.168.252.2`
- **Port**: `50000`
- **Authentication type**: `Username and password`
- **Username**: `db2inst1`
- **Password**: `db2inst1`
- **Port is SSL enabled**: `<toggled off>`


```bash
docker exec ibm-lh-postgres printenv | grep POSTGRES_PASSWORD | sed 's/.*=//'
```

- **Display name**: `PostgreSQLDB`
- **Database name**: `gosales`
- **Hostname**: `192.168.252.2`
- **Port**: `5432`
- **Username**: `admin`
- **Password**: `<The password generated in the previous step>`
- **Port is SSL enabled**: `<toggled off>`
- **Catalog name**: `pgcatalog`

```sql
select pll.product_line_en as product,
md.order_method_en as order_method,
sum(sf.quantity) as total
from
pgcatalog.gosalesdw.sls_order_method_dim as md,
db2catalog.gosalesdw.sls_product_dim as pd,
db2catalog.gosalesdw.sls
product
_
_line_lookup as pll,
hive_data.gosalesdw.sls_product_brand_lookup as pbl,
hive_data.gosalesdw.sls_sales_fact as sf
where
pd.product_key = sf.product_key
and md.order_method_key = sf.order_method_key
and pll.product_line_code = pd.product_line_code
and pbl.product_brand_code = pd.product_brand_code
group by pll.product_line_en, md.order_method_en
order by product, order_method;
```

```sql
select distinct branch_region_dim.region_en region,
branch_region_dim.country_en country,
emp_employee_dim.employee_name employee
from hive_data.gosalesdw.go_region_dim branch_region_dim,
pgcatalog.gosalesdw.emp_employee_dim emp_employee_dim,
db2catalog.gosalesdw.go_branch_dim go_branch_dim
where branch_region_dim.country_en in ('Canada', 'Mexico') and
branch_region_dim.country_code = go_branch_dim.country_code and
emp_employee_dim.branch_code = go_branch_dim.branch_code
order by region, country, employee;
```

```sql
select gosalesdw.go_org_dim.organization_key,
go_org_dim_1.organization_parent as org_level1_code,
go_org_name_lookup_1.organization_name_en as org_level1_name,
gosalesdw.go_org_dim.organization_parent as org_level2_code,
go_org_name_lookup_2.organization_name_en as org_level2_name,
gosalesdw.go_org_dim.organization_code as org_code,
gosalesdw.go_org_name_lookup.organization_name_en as org_name
from db2catalog.gosalesdw.go_org_name_lookup go_org_name_lookup_2
inner join
hive_data.gosalesdw.go_org_dim
inner join
pgcatalog.gosalesdw.go_org_name_lookup
on hive_data.gosalesdw.go_org_dim.organization_code =
pgcatalog.gosalesdw.go_org_name_lookup.organization_code
on go_org_name_lookup_2.organization_code =
hive_data.gosalesdw.go_org_dim.organization_parent
inner join
pgcatalog.gosalesdw.go_org_name_lookup go_org_name_lookup_1
inner join
hive_data.gosalesdw.go_org_dim go_org_dim_1
on go_org_name_lookup_1.organization_code =
go_org_dim_1.organization_parent
on hive_data.gosalesdw.go_org_dim.organization_parent =
go_org_dim_1.organization_code
where (hive_data.gosalesdw.go_org_dim.organization_code
between '1700' and '5730')
order by org_code;
```

11 Offloading Data to Object Storage
```bash
iceberg_data
```
```bash
wxgosalesdw
```
11.6
```sql
create table iceberg_data.wxgosalesdw.sls_sales_fact
as select * from db2catalog.gosalesdw.sls_sales_fact;
```

11.7
```sql
select pll.product_line_en as product,
sum(sf.quantity) as total
from
db2catalog.gosalesdw.sls_product_dim as pd,
db2catalog.gosalesdw.sls_product_line_lookup as pll,
iceberg_data.wxgosalesdw.sls_sales_fact as sf
where
pd.product_key = sf.product_key
and pll.product_line_code = pd.product_line_code
group by pll.product_line_en
order by product;
```

11.1.2
```sql
create table iceberg_data.my_schema.airport_id
as select * from hive_data.ontime.airport_id;
```

```sql
insert into iceberg_data.my_schema.airport_id values (10000, 'North Pole: Reindeer Field');
select * from iceberg_data.my_schema.airport_id where code = 10000;
select count(*) from iceberg_data.my_schema.airport_id;
```
```sql
call iceberg_data.system.rollback_to_snapshot ('my_schema', 'airport_id', <snapshotID>);
```

```sql
select * from iceberg_data.my_schema.airport_id where code = 10000;
select count(*) from iceberg_data.my_schema.airport_id;
```

11.2.2 Time Travel Queries
```sql
create table iceberg_data.my_schema.ttqtable (id int, name char(10));
```
```sql
select * from iceberg_data.my_schema."ttqtable$snapshots" order by committed_at;
```
```sql
insert into iceberg_data.my_schema.ttqtable values (1, 'TV'), (2, 'Computer'), (3, 'Stereo');
```
```sql
select * from iceberg_data.my_schema."ttqtable$snapshots" order by committed_at;
```
```sql
insert into iceberg_data.my_schema.ttqtable values (4, 'Phone'), (5, 'Tablet'), (6, 'Camera');
insert into iceberg_data.my_schema.ttqtable values (7, 'Headphones'), (8, 'Microphone'), (9, 'Watch');
```
```sql
select * from iceberg_data.my_schema."ttqtable$snapshots" order by committed_at;
```

11.2.10
```sql
select * from iceberg_data.my_schema.ttqtable for version as of 'Transaction1SnapshotID' order by id;
```
```sql
select * from iceberg_data.my_schema.ttqtable for timestamp as of cast('Tran1Time' as timestamp with time zone) order by id;
```
```sql
select * from iceberg_data.my_schema.ttqtable for version as of Tran2SnapshotID order by id;
```
```sql
select * from iceberg_data.my_schema.ttqtable for timestamp as of cast('Tran3Time' as timestamp with time zone) order by id;
```
```sql
select * from iceberg_data.my_schema.ttqtable for timestamp as of cast('Pick-a-time-between-Tran2Time-and-Tran3Time' as timestamp with time zone) order by id;
```
```sql
select * from iceberg_data.my_schema.ttqtable for timestamp as of cast('2023-01-01 00:00:00.000 UTC' as timestamp with time zone) order by id;
```
