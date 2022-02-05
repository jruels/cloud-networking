# Analyzing network traffic with VPC Flow Logs



## Overview

In this lab, you configure a network to record traffic to and from an Apache web server using VPC Flow Logs. You then export the logs to BigQuery to analyze them.

### Objectives

In this lab, you learn how to perform the following tasks:

- Configure a custom network with VPC Flow Logs
- Create an Apache web server
- Verify that network traffic is logged
- Export the network traffic to BigQuery to further analyze the logs



## Task 1. Configure a custom network with VPC Flow Logs

### Create the custom network

By default, VPC Flow Logs are disabled for a network. Therefore, you will create a new custom-mode network and enable VPC Flow Logs.

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **VPC network** > **VPC networks**.

* Click **Create VPC Network**.

* Specify the following, and leave the remaining settings as their defaults:

| Property    | Value (type value or select option as specified) |
| :---------- | :----------------------------------------------- |
| Name        | vpc-net                                          |
| Description | Enter an optional description                    |

* For **Subnet creation mode**, click **Custom**.

* Specify the following, and leave the remaining settings as their defaults:

| Property         | Value (type value or select option as specified) |
| :--------------- | :----------------------------------------------- |
| Name             | vpc-subnet                                       |
| Region           | us-central1                                      |
| IP address range | 10.1.3.0/24                                      |
| Flow Logs        | On                                               |

> Turning on VPC flow logs doesn't affect performance, but some systems generate a large number of logs, which can increase costs. If you click on **Configure logs** you'll notice that you can modify the aggregation interval and sample rate. This allows you to trade off longer interval updates for lower data volume generation which lowers logging costs. 

* Click **Done**.

* Click **Create**.

Wait for the network to be created before continuing.



### Create the firewall rule

In order to serve HTTP and SSH traffic on the network, you need to create a firewall rule.

* On the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **VPC network** > **Firewall**.

* Click **CREATE FIREWALL RULE**.

* Specify the following, and leave the remaining settings as their defaults:

| Property            | Value (type value or select option as specified) |
| :------------------ | :----------------------------------------------- |
| Name                | allow-http-ssh                                   |
| Network             | vpc-net                                          |
| Targets             | Specified target tags                            |
| Target tags         | http-server                                      |
| Source filter       | IP Ranges                                        |
| Source IP ranges    | 0.0.0.0/0                                        |
| Protocols and ports | Specified protocols and ports                    |

* For **tcp**, specify ports **80** and **22**.

Make sure to include the **/0** in the **Source IP ranges** to specify all networks.

* Click **Create**.



## Task 2. Create an Apache web server

### **Create the web server**

* On the **Navigation menu**, click **Compute Engine** > **VM instances**.

* Click **Create instance**.

* Specify the following, and leave the remaining settings as their defaults:

| Property     | Value (type value or select option as specified) |
| :----------- | :----------------------------------------------- |
| Name         | web-server                                       |
| Region       | us-central1                                      |
| Zone         | us-central1-c                                    |
| Series       | N1                                               |
| Machine type | f1-micro                                         |

* Click **Management, security, disks, networking, sole tenancy**.

* Click **Networking**.

* For **Network tags**, type **http-server**.

* For **Network interfaces**, click the pencil icon to edit.

* Specify the following, and leave the remaining settings as their defaults:

| Property   | Value (type value or select option as specified) |
| :--------- | :----------------------------------------------- |
| Network    | vpc-net                                          |
| Subnetwork | vpc-subnet                                       |

* Click **Done**.

* Click **Create**.



### Install Apache

Configure the VM instance that you created as an Apache web server, and overwrite the default web page.

* For **web-server**, click **SSH** to launch a terminal and connect.

* In the **web-server** SSH terminal, update the package index:

```bash
sudo apt-get update
```

* Install the apache2 package:

```bash
sudo apt-get install apache2 -y
```

* To create a new default web page by overwriting the default, run the following:

```
echo '<!doctype html><html><body><h1>Hello World!</h1></body></html>' | sudo tee /var/www/html/index.html
```

* Exit the SSH terminal:

```
exit
```





## Task 3. Verify that network traffic is logged

### **Generate network traffic**

* Return to the **VM instances** page in the Cloud Console.

* For **web-server**, click on the **External IP** to access the server.

> You should see the **Hello World!** welcome page that you configured. Alternatively, you can access the server in a new tab by navigating to http://*Enter the external IP Address*.



### **Find your IP address**

Find the IP address of the computer you are using. One easy way to do this is to go to a website that provides this address.

* Open a browser in a new tab.

* Go to [www.google.com](https://www.google.com/) and search for "what's my IP". It will either directly reply with your IP or give you a list of sites that perform this service.

* Ensure that the IP address only contains numerals (IPv4) and is not represented in hexadecimal (IPv6).

* Copy your IP address. It will be referred to as *YOUR_IP_ADDRESS*.



### **Access the VPC Flow Logs**

* In the Cloud Console, go to **Navigation menu** > **Logging** > **Logs Explorer**.

* In the **Log fields** panel, under **RESOURCE TYPE**, click **Subnetwork**. In the Query results pane, entries from the subnetwork logs appear.

* In the **Log fields** panel, under **LOG NAME**, click **compute.googleapis.com/vpc_flows**. In the Query results panel, entries from the VPC flow logs appear. If you do not see **compute.googleapis.com/vpc_flows**, wait a few minutes for this log type to show up.

* Click inside the Query preview box. It becomes the Query builder box.

* In the Query builder box, at the end of line 2, press **Enter** to create a new line.

* On line 3, enter *YOUR_IP_ADDRESS* and click **Run Query**.

> If you do not see the **vpc_flows** filter option or if there are no logs, you might have to wait a few minutes and refresh. If after a couple of minutes you still don't see the **vpc_flows** filter option, click on the **External IP** of the **web-server** a couple of times to generate more traffic and check back on the **vpc_flows** filter option.

* Expand one of the log entries.

* Within the entry, expand the **jsonPayload** and then expand the **connection**.



Which of the following fields does the connection contain?

* Source IP address
* Source port
* Destination IP address
* Destination port
* The IANA protocol number



You can explore other fields within the log entry before continuing to the next task.



## Task 4. Export the network traffic to BigQuery to further analyze the logs



### **Create an export sink**

* On the **Navigation menu**, click **Logging** > **Logs Explorer**. This will clear the other informaiton that you entered in the last task.

* Under the **RESOURCE TYPE**, click **Subnetwork**. In the Query results pane, entries for all available subnetworks appear.

* Under the **LOG NAME** filter, click **compute.googleapis.com/vpc_flows**. In the Query results pane, only the VPC flow log entries are shown.

* Select **Actions** > **Create Sink**.

* For the **Sink name**, type **bq_vpcflows**, and click **NEXT**.

* In the **Select sink service** drop-down list, select **BigQuery dataset**.

* In the **Select BigQuery dataset** drop-down list, select **Create new BigQuery dataset**.

* For the **Name** of the dataset, enter **bq_vpcflows** and click **CREATE DATASET**.

* Click **NEXT** twice.

* Click **CREATE SINK**.



### **Generate log traffic for BigQuery**

Now that the network traffic logs are being exported to BigQuery, generate more traffic by accessing the web-server several times. Using Cloud Shell, you can `curl` the IP address of the web-server several times.

* On the **Navigation menu**, click **Compute Engine** > **VM instances**.

* Note the **External IP** address for the **web-server** instance. It will be referred to as *EXTERNAL_IP*.

* Click **Activate Cloud Shell** (![Cloud Shell](https://cdn.qwiklabs.com/ep8HmqYGdD%2FkUncAAYpV47OYoHwC8%2Bg0WK%2F8sidHquE%3D)).

* If prompted, click **Continue**.

* Store the *EXTERNAL_IP* in an environment variable in Cloud Shell:

```
export MY_SERVER=<Enter the EXTERNAL_IP here>
```

* Access the web-server 50 times from Cloud Shell:

```
for ((i=1;i<=50;i++)); do curl $MY_SERVER; done
```



### **Visualize the VPC Flow Logs in BigQuery**

* In the Cloud Console, on the **Navigation menu**, click **BigQuery**.

* If prompted, click **Done**.

* In the left pane, expand the **bq_vpcflows** dataset to reveal the table. You might have to first expand the Project ID to reveal the dataset.

* Click on the name of the table. It should start with **compute_googleapis**.

* Click on **Details** under the **Table Details**.

* Copy the portion of the Table ID that is after the colon(:). It will be referred to as *TABLE_ID*.

> If you do not see the **bq_vpcflows** dataset or if it does not expand, wait and refresh the page.

* Add the following to the BigQuery **Editor** and replace **your_table_id** with *TABLE_ID* while retaining the accents (`) on both sides:

```sql
#standardSQL
SELECT
jsonPayload.src_vpc.vpc_name,
SUM(CAST(jsonPayload.bytes_sent AS INT64)) AS bytes,
jsonPayload.src_vpc.subnetwork_name,
jsonPayload.connection.src_ip,
jsonPayload.connection.src_port,
jsonPayload.connection.dest_ip,
jsonPayload.connection.dest_port,
jsonPayload.connection.protocol
FROM
`your_table_id`
GROUP BY
jsonPayload.src_vpc.vpc_name,
jsonPayload.src_vpc.subnetwork_name,
jsonPayload.connection.src_ip,
jsonPayload.connection.src_port,
jsonPayload.connection.dest_ip,
jsonPayload.connection.dest_port,
jsonPayload.connection.protocol
ORDER BY
bytes DESC
LIMIT
15
```

1. Click **Run**.

> If you get an error, ensure that you did not remove the **#standardSQL** part of the query. If it still fails, ensure that the TABLE_ID did not include the Project ID.



Which columns does the results table contain?

* Sum of bytes sent
* Source IP address and port
* Protocol
* Destination IP address and port
* Subnet name
* VPC name



### **Analyze the VPC Flow Logs in BigQuery**

The previous query gave you the same information that you saw in the Cloud Console. Now, you will change the query to identify the top IP addresses that have exchanged traffic with your **web-server**.

* Create a new query in the BigQuery **Editor** with the following and replace **your_table_id** with *TABLE_ID* while retaining the accents (`) on both sides:

```sql
#standardSQL
SELECT
jsonPayload.connection.src_ip,
jsonPayload.connection.dest_ip,
SUM(CAST(jsonPayload.bytes_sent AS INT64)) AS bytes,
jsonPayload.connection.dest_port,
jsonPayload.connection.protocol
FROM
`your_table_id`
WHERE jsonPayload.reporter = 'DEST'
GROUP BY
jsonPayload.connection.src_ip,
jsonPayload.connection.dest_ip,
jsonPayload.connection.dest_port,
jsonPayload.connection.protocol
ORDER BY
bytes DESC
LIMIT
15
```

* Click **Run**.



> The results table now has a row for each source IP and is sorted by the highest amount of bytes sent to the **web-server**. The top result should reflect your Cloud Shell IP address.



> Unless you accessed the **web-server** after creating the export sink, you will not see your machine's IP address in the table.



You can generate more traffic to the **web-server** from multiple sources and query the table again to determine the bytes sent to the server.



## Task 5. Review

You configured a VPC network, enabled VPC Flow Logs, and created a web server in that network. Then, you generated HTTP traffic to the web server, viewed the traffic logs in the Cloud Console, and analyzed the traffic logs in BigQuery.

There are multiple use cases for VPC Flow Logs. For example, you might use VPC Flow Logs to determine where your applications are being accessed from in order to optimize network traffic expense, to create HTTP load balancers to balance traffic globally, or to deny unwanted IP addresses with Cloud Armor.



## Task 6. Cleanup

* Delete all virtual machines