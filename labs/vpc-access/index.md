# Controlling Access to VPC Networks

## Overview 

In this lab, you create two nginx web servers and control external HTTP access to the web servers using tagged firewall rules. Then you explore IAM roles and service accounts.



### Objectives 

In this lab, you learn how to perform the following tasks:

- Create an nginx web server
- Create tagged firewall rules
- Create a service account with IAM roles
- Explore permissions for the Network Admin and Security Admin roles



# Task 1. Create the web servers 

Create two web servers (**blue** and **green**) in the **default** VPC network. Then install **nginx** on the web servers and modify the welcome page to distinguish the servers.

## Create the blue server 

Create the **blue** server with a network tag.

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Compute Engine** > **VM instances**.

* Click **Create**.

* Specify the following, and leave the remaining settings as their defaults:

| Property | Value (type value or select option as specified) |
| :------- | :----------------------------------------------- |
| Name     | blue                                             |
| Region   | *Choose a region*                                |
| Zone     | *Choose a zone*                                  |

* Click **Management, security, disks, networking, sole tenancy**.

* Click **Networking**.

* For **Network tags**, type **web-server**.

> Network tags are used by networks to identify which VM instances are subject to certain firewall rules and network routes. In the next task, you create a firewall rule to allow HTTP access for VM instances with the **web-server** tag. Alternatively, you could select the **Allow HTTP traffic** checkbox, which would tag this instance as **http-server** and create the tagged firewall rule for tcp:80 for you.

* Click **Create**.

## Create the green server 

Create the **green** server without a network tag.

* Click **Create instance**.

* Specify the following, and leave the remaining settings as their defaults:

| Property | Value (type value or select option as specified) |
| :------- | :----------------------------------------------- |
| Name     | green                                            |
| Region   | *Choose a region*                                |
| Zone     | *Choose a zone*                                  |

* Click **Create**.



### Install nginx and customize the welcome page

Install nginx on both VM instances and modify the welcome page to distinguish the servers.

* For **blue**, click **SSH** to launch a terminal and connect.

* Run the following command to install nginx:

```bash
sudo apt-get install nginx-light -y
```

* Run the following command to open the welcome page in the nano editor:

```bash
sudo nano /var/www/html/index.nginx-debian.html
```

* Replace the `<h1>Welcome to nginx!</h1>` line with `<h1>Welcome to the blue server!</h1>`.

* Press **CTRL+O**, **ENTER**, **CTRL+X**.

* Run the following command to verify the change:

```bash
cat /var/www/html/index.nginx-debian.html
```

The output should contain the following (**do not copy; this is example output**):

```
...
<h1>Welcome to the blue server!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
...
```

* Close the SSH terminal to **blue**:

```bash
exit
```

Repeat the same steps for the **green** server:

* For **green**, click **SSH** to launch a terminal and connect.

* Run the following command to install nginx:

```bash
sudo apt-get install nginx-light -y
```

* Run the following command, to open the welcome page in the nano editor:

```bash
sudo nano /var/www/html/index.nginx-debian.html
```

* Replace the `<h1>Welcome to nginx!</h1>` line with `<h1>Welcome to the green server!</h1>`.

* Press **CTRL+O**, **ENTER**, **CTRL+X**.

* Run the following command to verify the change:

```bash
cat /var/www/html/index.nginx-debian.html
```

The output should contain the following (**do not copy; this is example output**):

```
...
<h1>Welcome to the green server!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
...
```

* Close the SSH terminal to **green**:

```
exit
```



# Task 2. Create the firewall rule

Create the tagged firewall rule and test HTTP connectivity.



## Create the tagged firewall rule

Create a firewall rule that applies to VM instances with the **web-server** network tag.

* In the Cloud Console, on the **Navigation menu**, click **VPC network** > **Firewall**.

* Notice the **default-allow-internal** firewall rule.

> The **default-allow-internal** firewall rule allows traffic on all protocols/ports within the **default** network. You want to create a firewall rule to allow traffic from outside this network to only the **blue** server, by using the network tag **web-server**.

* Click **Create Firewall Rule**.

* Specify the following, and leave the remaining settings as their defaults:

| Property            | Value (type value or select option as specified) |
| :------------------ | :----------------------------------------------- |
| Name                | allow-http-web-server                            |
| Network             | default                                          |
| Targets             | Specified target tags                            |
| Target tags         | web-server                                       |
| Source filter       | IP Ranges                                        |
| Source IP ranges    | 0.0.0.0/0                                        |
| Protocols and ports | Specified protocols and ports                    |

* For **tcp**, specify port **80**.

Make sure to include the **/0** in the **Source IP ranges** to specify all networks.

* Click **Create**.

## Creat a test-vm

* In the Cloud Console, click **Activate Cloud Shell** (![Cloud Shell](https://cdn.qwiklabs.com/ep8HmqYGdD%2FkUncAAYpV47OYoHwC8%2Bg0WK%2F8sidHquE%3D)).

* If prompted, click **Continue**.

* Run the following command to display the available zones:

```bash
gcloud compute zones list
```

* Choose a zone from this list. It will be referred to as `[YOUR_ZONE]`.

* Run the following command to create a **test-vm** instance, replacing `[YOUR_ZONE]`:

```bash
gcloud compute instances create test-vm --machine-type=n1-standard-1 --subnet=default --zone=[YOUR_ZONE]
```



The output should look like this (**do not copy; this is example output**):

```
NAME     ZONE        MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
test-vm  us-central1-a  n1-standard-1                  10.142.0.4   35.237.134.68  RUNNING
content_copy
```

You can easily create VM instances from the Cloud Console or the gcloud command line.



## Test HTTP connectivity

From **test-vm** `curl` the internal and external IP addresses of **blue** and **green**.

* In the Cloud Console, on the **Navigation menu**, click **Compute Engine** > **VM instances**. Note the internal and external IP addresses of **blue** and **green**.

* For **test-vm**, click **SSH** to launch a terminal and connect.

* To test HTTP connectivity to **blue**'s internal IP, run the following command, replacing **blue**'s internal IP:

```bash
curl <Enter blue's internal IP here>
```

You should see the `Welcome to the blue server!` header.

* To test HTTP connectivity to **green**'s internal IP, run the following command, replacing **green**'s internal IP:

```bash
curl <Enter green's internal IP here>
```

You should see the `Welcome to the green server!` header.

> You can HTTP access both servers using their internal IP addresses. The connection on tcp:80 is allowed by the **default-allow-internal** firewall rule, because **test-vm** is on the same VPC network as the web servers (**default** network).

* To test HTTP connectivity to **blue**'s external IP, run the following command, replacing **blue**'s external IP:

```bash
curl <Enter blue's external IP here>
```

You should see the `Welcome to the blue server!` header.

* To test HTTP connectivity to **green**'s external IP, run the following command, replacing **green**'s external IP:

```
curl <Enter green's external IP here>
```

> This should not work!

* Press **CTRL+C** to stop the HTTP request.

> As expected, you can only HTTP access the external IP address of the **blue** server, because the **allow-http-web-server** only applies to VM instances with the **web-server** tag.

To verify the same behavior from your browser, open a new tab and navigate to `http://[External IP of server]`.



# Task 3. Explore the Network and Security Admin roles

Cloud IAM lets you authorize who can take action on specific resources, which give you full control and visibility to manage cloud resources centrally. The following roles are used in conjunction with single-project networking to independently control administrative access to each VPC network:

- **Network Admin:** Permissions to create, modify, and delete networking resources, except for firewall rules and SSL certificates.
- **Security Admin:** Permissions to create, modify, and delete firewall rules and SSL certificates.

Explore these roles by applying them to a service account, which is a special Google account that belongs to your VM instance instead of to an individual end user. You will authorize **test-vm** to use the service account to demonstrate the permissions of the **Network Admin** and **Security Admin** roles, instead of creating a new user.



## Verify current permissions

Currently, **test-vm** uses the Compute Engine default service account, which is enabled on all instances created by the `gcloud` command-line tool and the Cloud Console.

Try to list or delete the available firewall rules from **test-vm**.

* Return to the **SSH** terminal of the **test-vm** instance.

* Run the following command to try to list the available firewall rules:

```bash
gcloud compute firewall-rules list
```

The output should look like this (**do not copy; this is example output**):

```
ERROR: (gcloud.compute.firewall-rules.list) Some requests did not succeed:
 - Insufficient Permission
```

> This should not work!

* Run the following command to try to delete the **allow-http-web-server** firewall rule:

```bash
gcloud compute firewall-rules delete allow-http-web-server
```

* Enter **Y**, if asked to continue.

The output should look like this (**do not copy; this is example output**):

```
ERROR: (gcloud.compute.firewall-rules.delete) Could not fetch resource:
 - Insufficient Permission
```

> This should not work!

The **Compute Engine default service account** does not have the right permissions to allow you to list or delete firewall rules. The same applies to other users who do not have the right roles.



## Create a service account

Create a service account and apply the **Network Admin** role.

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **IAM & admin** > **Service accounts**. Notice the **Compute Engine default service account**.

* Click **Create service account**.

* Set the **Service account name** to **Network-admin**.

* Click **Create**.

* For **Select a role**, select **Compute Engine > Compute Network Admin**.

* Click **Continue**.

* Click **Done**.

> Optionally you could create and download a file that contains the private key. However, we can just stop **test-vm** and authorize it to use the service account.



## Authorize test-vm

Authorize **test-vm** to use the **Network-admin** service account.



* On the **Navigation menu**, click **Compute Engine** > **VM instances**.

* Click on the name of the **test-vm** instance to open the **VM instance details** page.

* Click **Stop** and confirm to **Stop** the instance. Wait for the instance to stop before proceeding to the next step.

> An instance must be stopped to edit its service account.

* Once the instance stopped, click **Edit**.

* Select **Network-admin** for **Service account**.

* Click **Save** and wait for the instance to update.

* Click **Start** and confirm to **Start** the instance.

* Return to the **Vm instances** page and wait for **test-vm** to change to a running state.

* Click **SSH** for the **test-vm** instance.



### verify permissions 

* Run the following command to try to list the available firewall rules:

```bash
gcloud compute firewall-rules list
```

The output should look like this (**do not copy; this is example output**):

```
NAME                    NETWORK  DIRECTION  PRIORITY  ALLOW     DENY
allow-http-web-server   default  INGRESS    1000      tcp:80
default-allow-icmp      default  INGRESS    65534     icmp
default-allow-internal  default  INGRESS    65534     all
default-allow-rdp       default  INGRESS    65534     tcp:3389
default-allow-ssh       default  INGRESS    65534     tcp:22
content_copy
```

> This should work!

* Run the following command to try to delete the **allow-http-web-server** firewall rule:

```bash
gcloud compute firewall-rules delete allow-http-web-server
```

* Enter **Y**, if asked to continue.

The output should look like this (**do not copy; this is example output**):

```
ERROR: (gcloud.compute.firewall-rules.delete) Could not fetch resource:
 - Required 'compute.firewalls.delete' permission for 'projects/[PROJECT_ID]/global/firewalls/allow-http-web-server'
```

> This should not work!

As expected, the **Network Admin** role has permissions to list but not modify/delete firewall rules.



## Update service account and verify permissions

Update the **Network-admin** service account by providing it with the **Security Admin** role.

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **IAM & admin** > **IAM**.

* Find the **Network-admin** account. Focus on the **Name** column to identify this account.

* Click on the pencil icon for the **Network-admin** account.

* Change **Role** to **Compute Engine > Compute Security Admin**.

* Click **Save**.

* Return to the **SSH** terminal of the **test-vm** instance.

* Run the following command to try to list the available firewall rules:

```bash
gcloud compute firewall-rules list
```

The output should look like this (**do not copy; this is example output**):

```
NAME                    NETWORK  DIRECTION  PRIORITY  ALLOW     DENY
allow-http-web-server   default  INGRESS    1000      tcp:80
default-allow-icmp      default  INGRESS    65534     icmp
default-allow-internal  default  INGRESS    65534     all
default-allow-rdp       default  INGRESS    65534     tcp:3389
default-allow-ssh       default  INGRESS    65534     tcp:22
content_copy
```

> This should work!

* Run the following command to try to delete the **allow-http-web-server** firewall rule:

```bash
gcloud compute firewall-rules delete allow-http-web-server
```

* Enter **Y**, if asked to continue.

The output should look like this (**do not copy; this is example output**):

```
Deleted [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00e186e4b1cec086/global/firewalls/allow-http-web-server].
```

> This should work!

> As expected, the **Security Admin** role has permissions to list and delete firewall rules.



## Verify the deletion of the firewall rule

Verify that you can no longer HTTP access the external IP of the **blue** server because you deleted the **allow-http-web-server** firewall rule.

* Return to the **SSH** terminal of the **test-vm** instance.

* To test HTTP connectivity to **blue**'s external IP, run the following command, replacing **blue**'s external IP:

```bash
curl <Enter blue's external IP here>
```

> This should not work!

* Press **CTRL+C** to stop the HTTP request.

> Provide the **Security Admin** role to the right user or service account to avoid any unwanted changes to your firewall rules!



# Task 4. Review

In this lab, you created two nginx web servers and controlled external HTTP access using a tagged firewall rule. Then you created a service account with first the **Network Admin** role and then the **Security Admin** role to explore the different permissions of these roles.

If your company has a security team that manages firewalls and SSL certificates and a networking team that manages the rest of the networking resources, grant the security team the **Security Admin** role and the networking team the **Network Admin** role.



# Task 5. Cleanup 

Delete all the virtual machines you created during the lab. 