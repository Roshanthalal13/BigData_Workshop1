## Introduction 
We are going to do The "Getting Started with Hadoop" Tutorial Exercise 1: Ingest and query relational data. In this Tutorial, We will be mainly working with HDFS, Sqoop, Impala and Hive.
  
##  HDFS
  HDFS is a key part of the many Hadoop Echo-system technologies. It supports the rapid transfer of data between compute nodes. HDFS takes in data, it breaks the information down into many Separate blocks and distribute then to different nodes in a cluster, thus enabling highly efficient parallel processing.

## SQOOP
  Sqoop allows easy import and export of data from structured data stores such as relational databases, enterprise, data warehouse, and NoSQL systems. It uses a connecter based architecture which supports plugins that provides connectivity to new external systems.

## IMPALA
 Imapla is a parallel processing query engine that sits on top of clustered systems like Apache Hadoop. It uses HDFS as its underlying storage. It is a real-time query engine and it is very fast in launching and processing queries in comparison to Hive Map-reduce.

## HIVE
  Hive is an ETL (extract, transform, Load) and Data warehousing tool developed on top of HDFS. It makes job easy for performing operations like Ad-hoc queries, Analysis of huge datasets, and Data encapsulation.


## Prerequisites
1. Have VM (Virtual Machine) installed in your Computer.
2. Have Cloudera Quick-start installed in your Computer.
3. Check you setting:
    - Increase you Base Memory to the highest (all the way to the end of red line, as far as possible).
    - Increase your Processor from one to two.
    - Change your Network setting to NAT.

## Pre-Tutorial Note
Turn on the Virtual Machine and Start Cloudera Quick-start. Once Cloudera Quick-start is running on your computer then go ahead and launch browser inside your Cloudera Quick-start. When you launch the browser you will see Cloudera Homepage and you will see Manager Node, Worker Node, and their IP addresses. Note the IP addresses down because you will need it in order to root your machine later. Then launch the Cloudera manager button which you will find on the top right side of your Cloudera Homepage.

Note: You will see a pop up box in the middle of the screen saying "Attempting to connect to Cloudera Manager" then follow the code below.

## Aggregate 
We are trying to force start into Cloudera manager because computers with low Base Memory and Processor will face interruption while launching Cloudera Manager.
## Step1:
...

$ sudo  /home/cloudera/cloudera-manager  --force  --express

...

Now you will able to see the Cloudera Manager Homepage and all running services such as HDFS, Sqoop, Yarn and others. Make sure Services are running properly and you can see a green light before every services.

We are now rooting into mysql to changes the database to retail_db.
## Step2:
...
$ mysql -u root -p
...

This line of command is used to display your databases running in you MySQL.
## Step3:
...
> show databases;
...

This line of command will allow you to change your database to "retail_db" and to use it.
## Step4:
...
> use retail_db;
...
Your database is changed to "retail_db". Now you can exit from MySQL using "ctrl+c".

We are back in the cloudera@quickstart. We have to launch a Sqoop job now using the given line of code:
## Step5:
...
$ sqoop import-all-tables \
  -m 1             
  --connect jdbc:mysql://quickstart:3306/retail_db \
  --username retail_dba -P \
  --compression-codec=snappy \
  --as-parquetfile \
  --warehouse-dir=/user/hive/warehouse \
  --hive-import

...
It will ask your Cloudera password, please put your Cloudera password.

Now you will see the command working on your machine. It will take few minutes to finish the command while your computer is working hard to map, reduce and transfer 1345 records into warehouse-dir in Hive.

Once you are done, you will see cloudera@quickstart.

The following command will root you to Master/Manager Node. 
## Step6:
ssh root@{{masternode ipaddress}}

Note: you will find the Master Node's IP address on the Cloudera Manager Homepage when you --force --express.


Once the command is complete we can confirm that our data was imported into HDFS:
## Step7:
...
[cloudera@quickstart ~]$ hadoop dfs -ls /user/hive/warehouse/


[cloudera@quickstart ~]$ hadoop dfs -ls /user/hive/warehouse/categories/
.....
![hadoopfscommand](https://user-images.githubusercontent.com/42624428/48044538-06a75e80-e152-11e8-80e3-ac1ead7bce06.PNG)


 

Now we have to login into Hue, which you will find out on the top left corner of your Cloudera HomePage. In order to log in Hue you have to use your Cloudera Username and Password. Once you are into Hue, click on Query editor and open the Impala Query Editor and you can copy/paste the code below:

## Step9:
...
invalidate metadata;

show tables;
...
![capture](https://user-images.githubusercontent.com/42624428/48044472-c1832c80-e151-11e8-8fc1-1170b6774542.PNG)

You can also click on "Refresh Table List" Icon on the left to see your new tables in the side menu.
After running the code in "Step9" you will see the name list of the tables you imported.

 


Now that we know our tables are imported and ready to use, we can run the following standard SQL code to display the Most Popular Product Categories.

## Step10:
.....
-- Most popular product categories
select c.category_name, count(order_item_quantity) as count
from order_items oi
inner join products p on oi.order_item_product_id = p.product_id
inner join categories c on c.category_id = p.product_category_id
group by c.category_name
order by count desc
limit 10;

.....


![capture2](https://user-images.githubusercontent.com/42624428/48044506-e5df0900-e151-11e8-8bf7-5d6a83cfbfb8.PNG)

This line of code will display top 10 revenue generating products.
 

## Step11:
.....

-- top 10 revenue generating products
select p.product_id, p.product_name, r.revenue
from products p inner join
(select oi.order_item_product_id, sum(cast(oi.order_item_subtotal as float)) as revenue
from order_items oi inner join orders o
on oi.order_item_order_id = o.order_id
where o.order_status <> 'CANCELED'
and o.order_status <> 'SUSPECTED_FRAUD'
group by order_item_product_id) r
on p.product_id = r.order_item_product_id
order by r.revenue desc
limit 10;

.....




![capture3](https://user-images.githubusercontent.com/42624428/48044525-f98a6f80-e151-11e8-96cd-67f6607980d5.PNG)

 






## Possible Error's You May Encounter

## Possible error 1:
Warning: /usr/lib/sqoop/ ../accumulo does not exist! Accumulo import will fail.
Please set $Accumulo_home to the root of your accumulo installation.

Note: When you encounter such error just follow the command given below.

You can choose either Solution ! or Solution 2.

## Solution1:
Step 1: create a "Sqoop" folder in you /usr/lib/    dir.

Step 2: Go to this https://accumulo.apache.org/downloads/   and download the suitable Accumulo package for your machine. (I installed older version which is version 1.7.2)

Note: open the .tar file and extract it to the "Sqoop" folder that you made previously.

Step 3: sudo yum install {copy and paste the extracted Accumulo file name including .rpm and hit enter}

Step 4: sudo mkdir /var/lib/accumulo

Step 5: ACCUMULO_HOME='/var/lib/accumulo'

Step 6: export ACCUMULO_HOME

Now you should be able to get through that error. 

## Solution 2:

Step 1: Login to Cloudera Managar and Click on the Hosts (Dropdown menu)

Step 2: Click on the Parcels Tab

You will see the list of packages including Accumulo.
Step 3: Click on Accumulo Download Tab.

Once the package is downloaded,

Step 4: Click on "Distribute" 

Step 5: Click "Activate"

NOw you shouldn't get that error again. And, you can install any other packages if needed.



## Possible error 2:
ERROR tool.Basesqooptool: Got error creating database manager: java.io.IOException: No Manager for connect string

## Solution:
This error means you have a command error, you are missing something in your command.
For example:
 --connect jdbc:mysql//{{cluster_data.manager_node_hostname}}:3306/retail_db \

 I was missing ':' in this code and i got that error. If you get this error too, then make sure you check and retype the correct command.











## Reference 
 https://www.youtube.com/watch?v=KuAKOOPbH00
 
 https://www.wikitechy.com/tutorials/sqoop/sqoop-job?fbclid=IwAR3pTOvrAYzSTFgs_PmWRzleYwcFVOW9SAIAfqm2d1TkDj4QhCOYwYG2c1c
 
 https://stackoverflow.com/questions/17194232/sqoop-import-multiple-tables?fbclid=IwAR3GQMgsI_BoZn9Ex0Qgdovt5cXCAA9ad8UTM1mWk1LZLYWT7U_NdpNN3KQ


