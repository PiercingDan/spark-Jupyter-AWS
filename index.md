# Spark with Jupyter on AWS
*By [Danny Luo](https://www.linkedin.com/in/danny-luo-277a93b8)*

A guide on how to set up Jupyter with Pyspark painlessly on AWS EC2 instances using the tool [Flintrock](https://github.com/nchammas/flintrock), with S3 I/O support. 

<img src="images/apachespark.jpg" alt="Drawing" width="280"/>
<img src="images/jupyter.png" alt="Drawing" width="280"/>
<img src="images/aws.png" alt="Drawing" width="280"/>


## Introduction
This guide was motivated by my involvement in the University of Toronto Data Science Team. The team recently undertook the Kaggle Competition: [Outbrain Click Prediction](https://www.kaggle.com/c/outbrain-click-prediction), in which one of the datasets was a large 88GB unzipped csv detailing the page views of users. To attempt to do any type of analysis with this particular dataset, we needed to use more powerful techniques. My fellow data enthuasiast and good friend, [Chris Goldsworthy](github.com/c4goldsw), suggested the idea of using Apache Spark, a fast big data parallel processing engine, to analyze this huge dataset. Chris and I had learned Spark on Databricks in preparation for Toronto's newest data hackathon [HackOn(Data)](http://hackondata.com/), at which we placed third for our project - [Optimal Digital Map Placement in Toronto](https://github.com/c4goldsw/billboardPlacementTO). We  wanted to showcase to the rest of our Data Science Team the power of distributed computing to effortlessly crunch large datasets without breaking the bank. We also wanted to set up clusters directly on the popular cloud computing platform, AWS, without using an intermediary tool like Databricks. 

The idea to use Jupyter Notebook with PySpark was due to my own affinity for Jupyter and python. Jupyter Notebook is highly effective for data science as it allows users to easily interact with their models and try many different approaches quickly. Jupyter also nicely embeds images, plots and tables for data visualization. Together with the numerous python libraries available, there is no end to the things you can do.

## Getting Started with AWS
AWS, short for [Amazon Web Services](https://aws.amazon.com/), is a popular cloud computing service. You will have to sign up for an account and have your credit card on hand. AWS does have a 1-year 'free' tier plan, which I am currently on, but it covers very limited services and I easily supercede the allotted amount and pay out of my own pocket. It is easy to rack up serious charges if you're not careful. This tutorial will use the cloud computing service EC2 and the cloud storage service S3. Try running a test cluster from AWS EC2 interface and uploading files onto a S3 bucket if you are not familiar with AWS.

For the following tutorial, you will need:
* [Amazon EC2 Key Pair](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) to log into your instances
* [Amazon Access Key ID and Secret Access Key](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSGettingStartedGuide/AWSCredentials.html) to use many programmatic features of AWS, such as integration of S3 in spark, and the AWS CLI
* Local Unix Environment

## Why Flintrock
There are many ways to set up spark with AWS EC2 instances, including:
* Manual Set-up
* [spark-ec2](https://github.com/amplab/spark-ec2)
* [Amazon EMR](https://aws.amazon.com/emr/)
* [**Flintrock**](https://github.com/nchammas/flintrock)

One could start EC2 instances and manually link the workers to the master. However, this is tedious and difficult to scale. I have used the spark-ec2 tool and although it gets the job done, the tool is not easy to use. It has slow launches for large clusters, clumsy command line integration without config file support, unresizable clusters and many other shortcomings. Flintrock, created by Nicholas Chammas, is the answer to spark-ec2's deficiencies. Like spark-ec2, it is also a command-line tool for launching Apache Spark clusters except it is easier to use and has more features. You can install it at the link above. This tutorial uses Flintrock 0.6.0.

Amazon EMR (Elastic MapReduce) is another option that I have not explored in depth. It seems like a solid service offered by Amazon to run and scale Hadoop/Spark and other Big Data frameworks. However, there is an [Amazon EMR cost](https://aws.amazon.com/emr/pricing/) in addition to the cost of the EC2 instances, which made me want to try to setup my own spark clusters using third-party tools at no extra expense.

## Using Flintrock

After installation, read the README in the Flintrock github page and try to run a couple of test clusters. After doing so, there are important modifications to make to the configuration file in Flintrock. 

I use the latest spark version (2.0.2 at the time of this tutorial) to harness the full capabilities of spark. However, there is a known issue with the incompatability of Hadoop 2.6/2.7 with S3 ([SPARK-7442](https://issues.apache.org/jira/browse/SPARK-7442)), and you will get an error when trying to use S3 in the spark interface. To fix this, we simply use Hadoop 2.4 and a Spark version built against Hadoop 2.4 available [here](https://spark.apache.org/downloads.html), for which S3 I/O works. The link below in `download-source` may be deprecated so check with the above website for any updates.

Open the configuration file and specify this as follows:

```
[LocalComputer] flintrock configure
```

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
    
(Rest of the file...)
```
All other parameters are up to you. I recommend using HVM AMI's rather than PVM AMI's as all instances on EC2 support HVM but not PVM. I also recommend using t2.micro instances, the simplest HVM instance, to practice the setup as they are covered in the free-tier. 

When finished, simply launch and log into your cluster through the Flintrock command line interface. I launch a custom AMI built from the Amazon Linux AMI. To begin, you may simply launch the default Amazon Linux AMI.

## Installing Anaconda/Jupyter
Installing Jupyter Notebook and its dependencies can be easily achieved by installing the Anaconda Python distribution. This will also include many important libraries such as numpy, scipy, matplotlib, pandas, etc. You can visit the installation page [here](https://www.continuum.io/downloads#). I download and install the Anaconda 4.2.0 Python 2.7 version while logged into my Spark master with the following commands: 

```
[SparkMaster] wget https://repo.continuum.io/archive/Anaconda2-4.2.0-Linux-x86_64.sh
[SparkMaster] sh Anaconda2-4.2.0-Linux-x86_64.sh
```

After installation, make sure you set your path to Anaconda properly as follows:
```
[SparkMaster] export PATH=/home/user/path/to/anaconda2/bin:$PATH
```
Better yet, place this in your .bashrc file, and save the AMI (detailed in a later section). Next time you may launch clusters using your saved custom AMI through Flintrock and your Anaconda path will be automatically set. 

Try running Jupyter notebook as a quick test.
```
[SparkMaster] jupyter notebook
```
Press Ctrl-C to exit.

## Using tmux (Optional)
You may have noticed in the previous command, or know from previous uses, that running jupyter notebook will hinder you from running any more commands in the same terminal. We want to be able to monitor the condition of PySpark driver while running other terminal commands. A simple workaround would be to open up another terminal and login a second time, but I prefer to use `tmux`, a useful tool that allows you split up your terminal. You can install it from the terminal, for example, on Amazon Linux, run:
```
[SparkMaster] sudo yum update
[SparkMaster] sudo yum install tmux
```
If you're running Ubuntu or a similar OS, replace `yum` with `apt-get`. 

Then, launch tmux with the command:
```
[SparkMaster] tmux new
```
You can familiarize yourself with the tmux commands [here](https://gist.github.com/MohamedAlaa/2961058).

## Launching Jupyter with PySpark
First, add the following lines in our `.bashrc` file at the end of the file.
```shell
#Amazon Keys
export AWS_ACCESS_KEY_ID=XXXXXXXXXXXXXXXXXXX
export AWS_SECRET_ACCESS_KEY=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# User specific aliases and functions #Note that your py4j Version may vary!
export PYTHONPATH=$SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.1-src.zip:$PYTHONPATH

# added by Anaconda2 4.2.0 installer
export PATH='/home/user/path/to/anaconda2/bin:$PATH'
```
The environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` will allow you to access your S3 buckets in spark. The rest of the commands set up necessary paths. Save your `.bashrc` file and source it.

Now, we are going to run Jupyter with Pyspark. The following steps are modified from this [tutorial](http://blog.insightdatalabs.com/jupyter-on-apache-spark-step-by-step/). 

I put the following commands in a shell script `jupyter_setup.sh`.

```shell
export spark_master_hostname=SparkMasterPublicDNS
export memory=1000M 

PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS="notebook --no-browser --port=7777" pyspark --master spark://$spark_master_hostname:7077 --executor-memory $memory --driver-memory $memory
```

You can find your Spark Master Public DNS on the AWS EC2 console. Note that each time you start a cluster, the Public DNS will be different, therefore you will need to alter it with the right DNS. To set the proper amoutn of memory, you can check how much memory there is available on your instances by going to *SparkMasterPublicDNS:/8080* in your browser. Note that Flintrock's default security group should have the port 8080 open to your computer by default; if not, then you can change it manually on the EC2 console. 

Source `jupyter_setup.sh`. By default, jupyter should run on port 7777. Then, to access the notebook on your browser, port forward on your local computer:
```
[LocalComputer] ssh -i ~/path/to/AWSkeypair.pem -N -f -L localhost:7776:localhost:7777 ec2-user@SparkMasterPublicDNS
```
The default user in Flintrock is `ec2-user` so that will work most of the time. I have found, annoyingly, that ubuntu instances will require `ubuntu@SparkMasterPublicDNS` instead.

Alternatively, you can configure your security group to allow your local computer to access the port directly. 

Now, go to your browser and type in *localhost:7776*. You should see the jupyter interface displaying the contents of the directory of your Spark Master in which you ran `jupyter_setup.sh`.

## Running Tests
Now, you can run some basic tests on jupyter to make sure everything is set up correctly. While you do so, you can observe your progress on built-in user interfaces at *SparkMasterPublicDNS:/8080* and *SparkMasterPublicDNS:/4040*. 

I have created a Jupyter notebook called `Spark_Test.ipynb` located in this [repository](https://github.com/PiercingDan/spark-Jupyter-AWS), which contains basic tests for Spark with S3 I/O. You may download this locally and then upload it onto your Spark Master through the Jupyter browser interface or directly download it from github onto your Spark Master. 

In addition, you will need to upload the test dataset `iris_data.csv` onto your S3 bucket. While you can do this through the S3 interface on AWS console, it is a worthwhile exercise to use the AWS CLI to perform this task. Download dataset on your Spark Master or your local computer with AWS CLI installed. Begin by setting up your AWS configurations:
```
[SparkMaster] aws configure
```
Download the Iris csv onto your Spark Master:
```
[SparkMaster] wget https://raw.githubusercontent.com/PiercingDan/spark-Jupyter-AWS/master/iris_data.csv
```
Upload it onto your S3 bucket (assuming you've already created one):
```
[SparkMaster] aws s3 cp iris_data.csv s3://BucketName/
[SparkMaster] aws s3 ls s3://BucketName
```
The AWS command line interface is a terrific tool, and it comes installed with all EC2 Instances. I use AWS CLI to quickly set up my code when logging onto a new Spark EC2 instance by running:
```
[SparkMaster] aws s3 sync s3://BucketName/code $HOME/code
```
When I'm finished working and ready to terminate by instances, I run the opposite sync to save my work to S3:
```
[SparkMaster] aws s3 sync $HOME/code s3://BucketName/code
```

## Using your own AMI
You wouldn't want to download and install Anaconda everytime you use spark, and you definitely want to terminate your instances after you're finished using them for cost reasons. The solution here is to save the AMI after running through this tutorial once, so the next time you launch a Spark cluster through Flintrock, you already have existing environment set up, i.e. Anaconda installed, AWS credentials set up, etc.

You can save your AMI using the AWS EC2 console. You can specify your custom AMI in the Flintrock configuration file, a very useful feature of Flintrock not available on spark-ec2. Note that since Flintrock by default installs and configures Spark on each node, it is important that you delete the Spark folder and other files before saving your AMI or you will encounter conflicts with Flintrock when booting from your custom AMI. 

Here's a useful script `clean.sh` to remove all conflicts with the Flintrock setup (thanks to Chris).

```shell
# A script that can be used to clean an AMI to avoid
# issues when Flintrock tries to launch a cluster from
# this AMI.  This should be executed before creating
# an AMI of this machine.

#remove id_rsa
rm -f $HOME/.ssh/id_rsa

#remove anything related to spark
rm -rf $HOME/spark

cd /usr/local/bin
rm -f *
```

It is also possible to tell Flintrock not to install Spark, if you ever should need to.

```
[LocalComputer] flintrock launch my-cluster --no-install-spark
```

## Shrinking EBS Volume (Optional)
Through my own experiences, the price of EBS volumes outweighs the price of spot-requested instances. The price for Amazon EBS gp2 volumes is [$0.10 per GB-month](https://aws.amazon.com/ebs/pricing/) for US East and since Flintrock sets its default minimum EBS root volume to be 30 GB, the EBS volumes costs about $0.10/day per instance or $0.004/hour per instance regardless of the instance type or AMI, comparable to the spot-requested m3.medium instance cost of ~$0.01/hour per instance. In addition, a 30 GB hard drive on every worker isn't really necessary for most Spark jobs, since Spark operations are done in-memory. If you follow this guide and use S3 I/O to read and write files, you will barely use your EBS storage. This section will detail how to shrink your EBS volumes to reduce unnecessary costs.

#### Shrinking your AMI 
If you created your AMI from the EC2 instance launched from Flintrock, the snapshot of the AMI, or the root EBS volume of the AMI, is 30 GB. Our first job is to shrink this volume. 

Although AWS provides an easy way to grow EBS volumes, it does not have a direct method to shrink them. There are many workarounds to shrink an EBS volume (simply google "Shrink EBS Volume" to see), but the general strategy is to create a running EBS volume of your AMI snapshot and a smaller EBS volume of your desired new size and copy the files from the former to the latter. Most methods offered by the community either don't work properly or are excessively complicated. Instead, I give a quick and easy solution (Much thanks to Chris for his help).

1. From the EC2 console, create an EBS volume (gp2) based on the snapshot of AMI you wish to shrink.
2. Create a new EC2 instance using the same OS as your AMI (Flintrock default is Amazon Linux AMI) with an EBS volume of your desired size. 
3. Attach the large AMI volume created in Step 1 to the instance. The default device name is `/dev/sdf`.
4. Login to your instance.
5. Run `lsblk` as a check to see all attached volumes. It should look like this (xvda1 refers to the first partition of xvda, the root volume):
    
    ```
[ec2-user@privateipaddress]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  10G  0 disk 
└─xvda1 202:1    0  10G  0 part /
xvdf    202:80   0  30G  0 disk 
└─xvdf1 202:81   0  30G  0 part 
```
6. Make a mount point `sudo mkdir /oldvol`, then mount the attached volume `sudo mount /dev/xvdf1 /oldvol`.
7. Copy over all the important files from old volume to current root volume `sudo rsync -aAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} /old/ /`. This will take a couple of minutes.
8. Unmount the volume `sudo umount /oldvol` and stop the instance.
9. Make an AMI of your instance, which is now your desired size. Test it out, using Flintrock.

This method is easier and more straightforward than others given by the community. Since we created our new EBS volume by starting an instance instead of creating an EBS volume from scratch, we do not have to worry about partioning the drive, implementing a file system or setting the root flag to make the volume bootable. We simply copy all important files from our old, large volume to our new, small volume.

#### Modifying Flintrock Source Code
As of Flintrock 0.7.0, the minimum root ebs volume of 30 GB is hardcoded into Flintrock (see [my issue](https://github.com/nchammas/flintrock/issues/174)). We have to change the source code of Flintrock in order to launch instances with EBS volume under 30 GB. Go to the Flintrock directory in your python site-packages folder and modify the argument `min_root_device_size_gb = 30` in `ec2.py` to your desired size.

Try running Flintrock again with your new AMI and you should see smaller EBS volumes created.

## Next Steps
The following are tasks that I am currently trying to figure out. They are not crucial but nice-to-have.

* How to read zip csv files from S3 as Spark DF or RDD without too much hassle
* ~~How to shrink EBS volume size launches, EBS costs are greatly outweighing spot-requested EC2 instances cost~~ See added section.

## Conclusion
Thanks for reading this guide! If this guide helped you, please give it a star on github. If you have any problems, create an issue.

## About Me
I am currently a student in Math & Physics at the University of Toronto, with a deep interest in data science, machine learning and big data. 

* [linkedin](https://www.linkedin.com/in/danny-luo-277a93b8)
* [github](https://github.com/PiercingDan)

## Acknowledgements
Many thanks to [Chris Goldsworthy](https://github.com/c4goldsw) for his help. 

## Sources
* http://blog.insightdatalabs.com/jupyter-on-apache-spark-step-by-step/
* https://github.com/nchammas/flintrock
