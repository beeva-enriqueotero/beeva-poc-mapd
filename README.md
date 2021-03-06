# beeva-poc-mapd
Proof of Concept with MapD at BEEVA


### Deploy on premise
- Use [MapD AMI (Community) from AWS Marketplace](https://aws.amazon.com/marketplace/pp/B071H71L2Y). *Version MapD Community Edition - v3-1e20d964-1d79-4290-9751-5af45b74f67b-ami-1128b507.4 (ami-8967fb9f)*
  - This AMI includes *mapd-ce-3.1.2-20170719-9b89178-Linux-x86_64-render*
  - We tested also on *mapd-ce-3.1.3-20170727-edbb780-Linux-x86_64-render*

*Note: instance ssh user is "centos" and not "ec2-user"*



### Quick start
Launch *mapdql*
```
read MAPD_PASSWORD
# MAPD_PASSWORD value is the instance id
export MAPD_PASSWORD
/raidStorage/prod/mapd/bin/mapdql -u mapd -p $MAPD_PASSWORD
# Type \h to get help
```

### Prepare benchmark
*Based on [Redshift benchmark tutorial](http://docs.aws.amazon.com/redshift/latest/dg/tutorial-tuning-tables-create-test-data.html)*

Edit `create_tables.sql`
```
sudo yum install -y nano
nano create_tables.sql
# go to http://docs.aws.amazon.com/redshift/latest/dg/tutorial-tuning-tables-create-test-data.html
# copy to cliboard, paste and save file
```
Create tables
```
/raidStorage/prod/mapd/bin/mapdql -u mapd -p $MAPD_PASSWORD < create_tables.sql
```

Install aws cli to access to S3 files
```
sudo yum install -y python-pip
pip install --upgrade --user awscli
```

Ingest data directly from S3 (Slower alternative. 15 mins each 75M rows)
```
# Remember to attach a role to your instance to have access to s3
aws s3 cp s3://awssampledbuswest2/ssbgz/customer0002_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert customer mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
aws s3 cp s3://awssampledbuswest2/ssbgz/dwdate.tbl.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert dwdate mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0000_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert lineorder mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0001_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert lineorder mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0002_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert lineorder mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0003_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert lineorder mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0004_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert lineorder mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0005_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert lineorder mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0006_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert lineorder mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0007_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert lineorder mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/part0000_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert part mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/part0001_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert part mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/part0002_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert part mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
time aws s3 cp s3://awssampledbuswest2/ssbgz/part0003_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert part mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
```


Ingest data by copying files locally (Faster alternative. 4 mins each 75M rows)
```
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0000_part_00.gz .
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0001_part_00.gz .
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0002_part_00.gz .
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0003_part_00.gz .
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0004_part_00.gz .
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0005_part_00.gz .
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0006_part_00.gz .
time aws s3 cp s3://awssampledbuswest2/ssbgz/lineorder0007_part_00.gz .
time gunzip -d lineorder0000_part_00.gz
time gunzip -d lineorder0001_part_00.gz
time gunzip -d lineorder0002_part_00.gz
time gunzip -d lineorder0003_part_00.gz
time gunzip -d lineorder0004_part_00.gz
time gunzip -d lineorder0005_part_00.gz
time gunzip -d lineorder0006_part_00.gz
time gunzip -d lineorder0007_part_00.gz
/raidStorage/prod/mapd/bin/mapdql -u mapd -p $MAPD_PASSWORD
COPY lineorder from '/home/centos/lineorder0000_part_00' WITH (delimiter = '|', header = 'false') ;
COPY lineorder from '/home/centos/lineorder0001_part_00' WITH (delimiter = '|', header = 'false') ;
COPY lineorder from '/home/centos/lineorder0002_part_00' WITH (delimiter = '|', header = 'false') ;
COPY lineorder from '/home/centos/lineorder0003_part_00' WITH (delimiter = '|', header = 'false') ;
COPY lineorder from '/home/centos/lineorder0004_part_00' WITH (delimiter = '|', header = 'false') ;
COPY lineorder from '/home/centos/lineorder0005_part_00' WITH (delimiter = '|', header = 'false') ;
COPY lineorder from '/home/centos/lineorder0006_part_00' WITH (delimiter = '|', header = 'false') ;
COPY lineorder from '/home/centos/lineorder0007_part_00' WITH (delimiter = '|', header = 'false') ;
# In case no free space on disk rm -rf lineorder000i_part_00 will be required after each copy
```
### Execute benchmark

See http://docs.aws.amazon.com/redshift/latest/dg/tutorial-tuning-tables-test-performance.html
use \timing to measure times

### Results

* Query 1: `revenue = 32879160652772` 
  * 1st request: 5.0+-0.5s 
  * next requests: 0.35+-0.05s 

* Query 2: `exception: Query would require a scan without a limit on table(s): lineorder, dwdate` or `TSocket::write_partial() send() <Host: localhost Port: 9091>Broken pipe`

* Query 3: `Exception: SlabTooBig`, after 52s of execution, or `TSocket::write_partial() send() <Host: localhost Port: 9091>Broken pipe`

*Notes*: 
- First execution of query 1 was much slower, 60s aprox.
- In none of these cases `nvidia-smi` shows Volatile GPU-util > 0%
- Tables are stored in ephemeral disk. So database could be lost after reboot.
- Query 2 is forbidden by watchdog. Watchdog can be disabled with `enable_watchdog = false` in `/raidStorage/prod/mapd-storage/mapd.conf`


### Restart mapd server
```
export MAPD_PATH=/raidStorage/prod/mapd
cd $MAPD_PATH
sudo systemctl stop mapd_server
sudo systemctl start mapd_server
```
### Results

- I was not able to run queries 2 and 3 even with `enable_watchdog = false`

### Issues
 - https://community.mapd.com/t/slabtoobig-exception/349
 - https://community.mapd.com/t/select-field-count-from-table-group-by-field-the-result-is-confuse/151

### Additional comments
- Many broken links in documentation
- Failed installation on [Deep Learning Ubuntu 14.04 AMI](https://aws.amazon.com/marketplace/pp/B06VSPXKDX) 
following [install guidelines](https://github.com/mapd/mapd-core#ubuntu-1604-1610). Error: Unable to install `clang-format`
- Mapd Benchmark tutorial vs Redshift and others: http://tech.marksblogg.com/benchmarks.html (howto: http://tech.marksblogg.com/billion-nyc-taxi-rides-aws-ec2-p2-8xlarge-mapd.html)

