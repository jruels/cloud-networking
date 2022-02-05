# Resource Monitoring



## Overview

In this lab, you learn how to use Cloud Monitoring to gain insight into applications that run on Google Cloud.



### Objectives

In this lab, you learn how to perform the following tasks:

- Explore Cloud Monitoring
- Add charts to dashboards
- Create alerts with multiple conditions
- Create resource groups
- Create uptime checks



# Task 1: Create a Cloud Monitoring workspace



## Verify resources to monitor

Three VM instances have been created for you that you will monitor.

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Compute Engine** > **VM instances**. Notice the **nginxstack-1**, **nginxstack-2** and **nginxstack-3** instances.



## Create a Monitoring workspace

You will now setup a Monitoring workspace that's tied to your Qwiklabs GCP Project. The following steps create a new account that has a free trial of Monitoring.

* In the Google Cloud Platform Console, click on **Navigation menu** > **Monitoring**.
* Wait for your workspace to be provisioned.

When the Monitoring dashboard opens, your workspace is ready.

<img src="https://cdn.qwiklabs.com/58FQA3ZYeF1Uh01sWQnh5ymX4wPAAjryBAbPh92DHsY%3D" alt="img" style="zoom:50%;" />



# Task 2: Custom dashboards



## Create a dashboard

* In the left pane, click **Dashboards**.
* Click **+Create Dashboard**.
* For **New Dashboard Name**, type **My Dashboard**.



## Add a chart

* From **Chart library**, Select **Line**.
* For **Title**, give your chart a name (you can revise this before you save based on the selections you make).
* For **Resource type**, select **VM Instance**.
* For **Metric**, select a metric to chart for the Instance resource, such as **CPU utilization** or **CPU usage**.
* Click **+ Add Filter** and explore the various options.



## Metrics Explorer

The **Metrics Explorer** allows you to examine resources and metrics without having to create a chart on a dashboard. Try to recreate the chart you just created using the **Metrics Explorer**.

* In the left pane, click **Metrics explorer**.
* For **Resource type** and **Metric**, select a resource name and metric.
* Explore the various options and try to recreate the chart you created earlier.

> Not all metrics are currently available on the Metrics Explorer, so you might not be able to find the exact metric you used on the previous step.



# Task 3: Alerting policies



## Create an alert and add the first condition

* In the left pane, click **Alerting**.
* Click **+ Create Policy**.
* Click **Add Condition**.
* For **Find resource type and metric**, select **VM Instance**.

> If you cannot locate the **VM Instance** resource type, you might have to refresh the page.

* Select a metric you are interested in evaluating, such as **CPU usage** or **CPU Utilization**.
* Under **Configuration**, for **Condition**, select **is above**.
* Specify the threshold value and for how long the metric must cross this set value before the alert is triggered. For example, for **THRESHOLD**, type **20** and set **FOR** to **1 minute**.
* Click **ADD**.



## Add a second condition

* Click **Add Condition**.
* Repeat the steps above to specify the second condition for this policy. For example, repeat the condition for a different instance. Click **ADD**.
* In **Policy Triggers**, for **Trigger when**, click **All conditions are met**.
* Click **Next**.



## Configure notifications and finish the alerting policy

* Click on dropdown arrow next to **Notification Channels**, then click on **Manage Notification Channels**.

A **Notification channels** page will open in new tab.

* Scroll down the page and click on **ADD NEW** for **Email**.
* Enter your personal email in the **Email Address** field and a **Display name**.
* Click **Save**.
* Go back to the previous **Create alerting policy** tab.
* Click on **Notification Channels** again, then click on the **Refresh icon** to get the display name you mentioned in the previous step. Click **Notification Channels** again if needed.
* Now, select your **Display name** and click **OK**.
* Click **Next**.
* Enter a name of your choice in **Alert name** field.
* Click **Save**.



# Task 4: Resource Groups

* In the left pane, click **Groups**.
* Click **+ Create Group**.
* Enter a name for the group. For example: **VM instances**
* In the **Criteria** section, type **nginx** in the value field below **Contains**.
* Click **DONE**.
* Click **CREATE**.
* Review the dashboard Cloud Monitoring created for your group.



# Task 5: Uptime monitoring

* In the Monitoring tab, click on **Uptime Checks**.
* Click **+ Create Uptime Check**.
* Specify the following, and leave the remaining settings as their defaults:

| Property            | Value (type value or select option as specified) |
| :------------------ | :----------------------------------------------- |
| **Title**           | *Enter a title* then click Next                  |
| **Protocol**        | **HTTP**                                         |
| **Resource Type**   | **Instance**                                     |
| **Applies To**      | **Group**                                        |
| **Group**           | *Select your group*                              |
| **Check Frequency** | **1 minute**                                     |

1. Click on **Next** to leave the other details to default. Under **Alert & Notification**, select your Notification Channels from the dropdown.
2. Click **Test** to verify that your uptime check can connect to the resource.
3. When you see a green check mark everything can connect. Click **Create**.

The uptime check you configured takes a while for it to become active.



# Task 6: Disable the alert

Disable the alert Alerting policies stay active for a while after a project is deleted, just in case it needs to be reinstalled. Since this is a lab, and you will not have access to this project again, remove the alerting policy you created.

You should still be in the **Alerting** section.

From you alert's Policy details page, click the **Enabled** link at the top of the page.

You will be asked to confirm that you want to disable the alerting policy - click **Disable**.

The link will now say Disabled.



# Task7: Review 

In this lab, you learned how to:

- Monitor your projects
- Create a Cloud Monitoring workspace
- Create alerts with multiple conditions
- Add charts to dashboards
- Create resource groups
- Create uptime checks for your services







