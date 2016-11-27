# Spark with Jupyter on AWS
*By Danny Luo*

A guide on how to set up Jupyter with Pyspark painlessly on AWS EC2 instances using the tool [Flintrock](https://github.com/nchammas/flintrock), with S3 I/O support. This guide will be updated as I discover more things.

## Introduction
This guide was motivated by my involvement in the University of Toronto Data Science Team. The team recently undertook the Kaggle Competition: [Outbrain Click Prediction](https://www.kaggle.com/c/outbrain-click-prediction), in which one of the datasets was a large 88GB unzipped csv detailing the page views of users. To attempt to do any type of analysis with this particular dataset, it was obvious that we had use more powerful machines. My fellow big data enthuasiast and good friend, [Chris Goldsworthy](github.com/c4goldsw), suggested the idea of using Apache Spark, a fast big data parallel processing engine, to analyze this huge dataset. Chris and I had learned Spark on Databricks in preparation for Toronto's newest data hackathon [HackOn(Data)](http://hackondata.com/), at which we placed third for our project-[Optimal Digital Map Placement in Toronto](https://github.com/c4goldsw/billboardPlacementTO). We decided that we wanted to showcase to the rest of our Data Science Team the power of distributed computing to effortlessly crunch large datasets. We also wanted to set up clusters directly on cloud computing platforms without using an intermediary tool like Databricks. We decided on AWS at it seemed to be the most popular and supported platform available.

The idea to use Jupyter Notebook with PySpark was due to my own affinity for Jupyter and python. I prefer to use Jupyter Notebook for data science as it allows users to interact quickly with their models and try many different approaches efficiently, which is important when starting out. Jupyter functionality also nicely embeds images for data visualization, something which is absolutely essential when working with data. I also really enjoyed using the notebook interface in Databricks when I was learning Spark. 

## Getting Started with AWS
AWS, short for [Amazon Web Services](https://aws.amazon.com/), is a popular cloud computing services. You will have to sign up for an account and have your credit card handy. AWS does have a 1-year 'free' tier plan, which I am currently on, but it covers very limited services and I easily supercede the allotted amount and pay out of my own pocket. It is easy to rack up serious charges if you're not careful. This tutorial will use the cloud computing service EC2 and the cloud storage service S3. Try running a test cluster from AWS EC2 interface and uploading files onto a S3 bucket if you are not familiar with AWS.

For the following tutorial, you will need:
* [Amazon EC2 Key Pair](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) to log into your instances
* [Amazon Access Key ID and Secret Access Key](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSGettingStartedGuide/AWSCredentials.html) to use many programmatic features of AWS, such as integration of S3 in spark, and the AWS CLI. 

## Why Flintrock
There are many ways to set up spark with AWS EC2 instances, including:
* Manual Set-up
* [spark-ec2](https://github.com/amplab/spark-ec2) tool
* [Amazon EMR](https://aws.amazon.com/emr/)
* [**Flintrock**](https://github.com/nchammas/flintrock)

One could do it manually, by starting instances first and linking the workers to the master. However, this is tedious and difficult to scale. I have used the spark-ec2 tool and although it gets the job done, the tool is not easy to use. It has slow launches for large clusters, clumsy command line integration without config file support, unresizable clusters and many other shortcomings. Flintrock, created by Nicholas Chammas, is the answer to spark-ec2's deficiencies. Like spark-ec2, it is also a command-line tool for launching Apache Spark clusters except it is easier to use and has more features. You can install it at the link above. The tutorial uses Flintrock 0.6.0.

Amazon EMR (Elastic MapReduce) is another option that I have not explored in depth. It seems like a solid service offered by Amazon to run and scale Hadoop/Spark and other Big Data frameworks. However, there is an [Amazon EMR cost](https://aws.amazon.com/emr/pricing/) in addition to the cost of the EC2 instances, which made me want to try to setup my own spark clusters using third-party tools.

## Using Flintrock

After installation, read the README in the flintrock github page and try to run a couple of test clusters. After doing so, there are important modifications to make to the configuration file in flintrock. 

We generally try to use the latest spark version (2.0.2 at the time of this tutorial) to harness the full capabilities of spark. However, there is a known issue with the incompatability of Hadoop 2.6 (and 2.7 as I attempted) with S3 ([SPARK-7442](https://issues.apache.org/jira/browse/SPARK-7442)), and you will get an error when trying to use S3 in the spark interface. To fix this, we simply use Hadoop 2.4 and a Spark version built against Hadoop 2.4 available [here](https://spark.apache.org/downloads.html), for which S3 I/O works. You should specify this in the configuration file as follows:

```yaml
services:
  spark:
    version: 2.0.2
    # git-commit: latest  # if not 'latest', provide a full commit SHA; e.g. d6dc12ef0146ae409834c78737c116050961f350
    # git-repository:  # optional; defaults to https://github.com/apache/spark
    # optional; defaults to download from from the official Spark S3 bucket
    #   - must contain a {v} template corresponding to the version
    #   - Spark must be pre-built
    #   - must be a tar.gz file
    download-source: "http://d3kbcqa49mib13.cloudfront.net/spark-2.0.2-bin-hadoop2.4.tgz"

  hdfs:
    version: 2.4 #2.7.2
    # optional; defaults to download from a dynamically selected Apache mirror
    #   - must contain a {v} template corresponding to the version
    #   - must be a .tar.gz file
    # download-source: "https://www.example.com/files/hadoop/{v}/hadoop-{v}.tar.gz"
```
All other parameters are up to you. I recommend using HVM AMI's as they have better support than PVM AMI's on EC2. I also recommend using t2.micro instances when trying to figure out all this setup as they are covered in the free-tier.

When finished, simply launch and log into your cluster through the flintrock command line interface. I launch a custom AMI built from the Amazon Linux AMI.

## Installing Anaconda/Jupyter
Installing Jupyter Notebook and its dependencies can be easily achieved by installing the Anaconda Python distribution. This will also include many important libraries such as numpy, scipy, matplotlib, pandas, etc. You can visit the installation page [here](https://www.continuum.io/downloads#). I download and install the Anaconda 4.2.0 Python 2.7 version while logged into my Spark master with the following commands: 

```
[SparkMasterPublicDNS] wget https://repo.continuum.io/archive/Anaconda2-4.2.0-Linux-x86_64.sh
[SparkMasterPublicDNS] sh Anaconda2-4.2.0-Linux-x86_64.sh
```

After installation, make sure you set your path to Anaconda properly as follows:
```
[SparkMasterPublicDNS]
```
Better yet, place this in your .bashrc file, and save the AMI (detailed in a later section). Next time you may launch clusters using your saved custom AMI through flintrock and your Anaconda path will be automatically set. 

## Running

## S3 Input/Output

## Using your own AMI
You wouldn't want to download and install Anaconda everytime. 

## Next Steps
The following are tasks that I am currently trying to figure out. They are not crucial but nice-to-have.

* How to read zip csv files from S3 as Spark DF or RDD without too much hassle
* How to shrink EBS volume size launches, EBS costs are greatly outweighing spot-requested EC2 instances cost

## Acknowledgements
Many thanks to [Chris Goldsworthy](www.github.com/c4goldsw) for his help. 

## Sources
* http://blog.insightdatalabs.com/jupyter-on-apache-spark-step-by-step/
