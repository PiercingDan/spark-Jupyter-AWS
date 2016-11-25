# Spark with Jupyter on AWS
*By Danny Luo*

A guide on how to set up Jupyter with Pyspark painlessly on AWS EC2 instances using the tool [Flintrock](https://github.com/nchammas/flintrock), with S3 I/O support. This guide will be updated as I discover more things.

## Introduction
This guide was motivated by my involvement in the University of Toronto Data Science Team. The team recently undertook the Kaggle Competition: [Outbrain Click Prediction](https://www.kaggle.com/c/outbrain-click-prediction), in which one of the datasets was a large 88GB unzipped csv detailing the page views of users. To attempt to do any type of analysis with this particular dataset, it was obvious that we had use more powerful machines. My fellow big data enthuasiast and good friend, [Chris Goldsworthy](github.com/c4goldsw), suggested the idea of using Apache Spark, a fast big data parallel processing engine, to analyze this huge dataset. Chris and I had learned Spark on Databricks in preparation for Toronto's newest data hackathon [HackOn(Data)](http://hackondata.com/), at which we placed third for our project-[Optimal Digital Map Placement in Toronto](https://github.com/c4goldsw/billboardPlacementTO). We decided that we wanted to showcase to the rest of our Data Science Team the power of distributed computing to effortlessly crunch large datasets. We also wanted to set up clusters directly on cloud computing platforms without using an intermediary tool like Databricks. We decided on AWS at it seemed to be the most popular and supported platform available.

The idea to use Jupyter Notebook with PySpark was related to my own affinity for Jupyter and python. I prefer to use Jupyter Notebook for data science as it allows users to interact quickly with their models and try many different approaches efficiently, which is important when starting out. Jupyter functionality also nicely embeds images for data visualization, something which is absolutely essential when working with data. 

## Getting Started with AWS
AWS, short for [Amazon Web Services](https://aws.amazon.com/), is a popular cloud computing services. You will have to sign up for an account and have your credit card handy. AWS does have a 1-year 'free' tier plan, which I am currently on, but it covers very limited services and I easily supercede the allotted amount and pay out of my own pocket. It is easy to rack up serious charges if you're not careful. 

Try running a test cluster from AWS EC2 page and seeing what you need. For the following tutorial, you will need:
* [Amazon EC2 Key Pair](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) to log in to your instances
* [Amazon Access Key ID and Secret Access Key](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSGettingStartedGuide/AWSCredentials.html) to use Amazon's SQS

## Why Flintrock

## Using Flintrock

## Installing Anaconda/Jupyter

## Running

## S3 Input/Output

## Using your own AMIs

## Unresolved Problems

## Acknowledgements
Much thanks to [Chris Goldsworthy](github.com/c4goldsw) for his help. 
