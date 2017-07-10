# beeva-poc-mapd
Proof of Concept with MapD at BEEVA


### Deploy on premise
- Use [MapD AMI (Community) from AWS Marketplace](https://aws.amazon.com/marketplace/pp/B071H71L2Y)

*Note: instance ssh user is "centos" and not "ec2-user"*

### Prepare benchmark
Edit `create_tables.sql`
```
sudo yum install -y nano
nano create_tables.sql
# go to http://docs.aws.amazon.com/redshift/latest/dg/tutorial-tuning-tables-create-test-data.html
# copy to cliboard, paste and save file
```
Create tables
```
read MAPD_PASSWORD
# MAPD_PASSWORD value is the instance id
export MAPD_PASSWORD
/raidStorage/prod/mapd/bin/mapdql -u mapd -p $MAPD_PASSWORD < create_tables.sql
```

Install aws cli to access to S3 files
```
sudo yum install -y python-pip
pip install --upgrade --user awscli
```

Ingest data
```
# Remember to attach a role to your instance to have access to s3
aws s3 cp s3://awssampledbuswest2/ssbgz/customer0002_part_00.gz - | gzip -d | /raidStorage/prod/mapd/SampleCode/StreamInsert customer mapd -u mapd -p $MAPD_PASSWORD --delim '|' --batch 100000 --print_error
```

### Additional comments
- Many broken links in documentation
- Failed installation on [Deep Learning Ubuntu 14.04 AMI](https://aws.amazon.com/marketplace/pp/B06VSPXKDX) 
following [install guidelines](https://github.com/mapd/mapd-core#ubuntu-1604-1610). Error: Unable to install `clang-format`
- Mapd Benchmark tutorial vs Redshift and others: http://tech.marksblogg.com/benchmarks.html (howto: http://tech.marksblogg.com/billion-nyc-taxi-rides-aws-ec2-p2-8xlarge-mapd.html)
- Redshift benchmark Tutorial: http://docs.aws.amazon.com/redshift/latest/dg/tutorial-tuning-tables-create-test-data.html
