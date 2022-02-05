# Caching Cloud Storage content with Cloud CDN



## Overview

In this lab, you configure Cloud Content Delivery Network (Cloud CDN) for a Cloud Storage bucket and verify caching of an image. Cloud CDN uses Google's globally distributed edge points of presence to cache HTTP(S) load-balanced content close to your users. Caching content at the edges of Google's network provides faster delivery of content to your users while reducing serving costs.

For an up-to-date list of Google's Cloud CDN cache sites, see https://cloud.google.com/cdn/docs/locations.



## Objectives



In this lab, you learn how to perform the following tasks:

- Create and populate a Cloud Storage bucket
- Create an HTTP load balancer with Cloud CDN
- Verify the caching of your bucket's content
- Invalidate the cached content



# Task 1. Create and populate a Cloud Storage bucket

Cloud CDN content can originate from different types of backends:

- Compute Engine virtual machine (VM) instance groups
- Zonal network endpoint groups (NEGs)
- Internet network endpoint groups (NEGs), for endpoints that are outside of Google Cloud (also known as custom origins)
- Google Cloud Storage buckets

In this lab, you configure a Cloud Storage bucket as the backend.



## **Create a unique Cloud Storage bucket**

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Cloud Storage > Browser**.

* Click **Create bucket**.

* Specify the following, and leave the remaining settings as their defaults:

  | Property              | Value (type value or select option as specified)             |
  | :-------------------- | :----------------------------------------------------------- |
  | Name                  | *Enter a globally unique name*                               |
  | Location type         | Region                                                       |
  | Location              | *Choose a location that is very far from you*                |
  | Prevent public access | Deselect option to Enforce public access prevention to the bucket |

Try to choose a location that is either halfway around the world from you or at least in a different continent. This provides a greater time difference between accessing the image with and without Cloud CDN enabled. See the current list of regions [here](https://cloud.google.com/about/locations#regions).

* Click **Create**.

Note the name of your storage bucket for the next subtask. It will be referred to as `[your-storage-bucket]`.



## **Copy image files into your bucket**

Copy images from a public Cloud Storage bucket to your own bucket.

* In the Cloud Console, click **Activate Cloud Shell** (![Cloud Shell](https://cdn.qwiklabs.com/ep8HmqYGdD%2FkUncAAYpV47OYoHwC8%2Bg0WK%2F8sidHquE%3D)).
* If prompted, click **Continue**.
* Store `[your-storage-bucket]` in an environment variable in Cloud Shell:

```
export YOUR_BUCKET=<Enter the name of [your-storage-bucket]>
```

* To copy two images from a public bucket to your bucket, run the following commands:

```
gsutil cp gs://cloud-training/gcpnet/cdn/cdn.png gs://$YOUR_BUCKET
gsutil cp gs://cloud-training/gcpnet/cdn/regions.png gs://$YOUR_BUCKET
```

* In the Cloud Console, click **Refresh** to verify that the images were copied.

* In Cloud Shell, to publish the image files to the web, run the following command:

```
gsutil iam ch allUsers:objectViewer gs://$YOUR_BUCKET
```

* In the Cloud Console, click **Refresh** to verify that both images have **Public access** set to **Public to internet**.



# Task 2. Create the HTTP load balancer with Cloud CDN



HTTP(S) load balancing provides global load balancing for HTTP(S) requests of static content to a Cloud Storage bucket (backend). When you enable Cloud CDN on your backend, your content is cached at a [location at the edge of Google's network](https://cloud.google.com/cdn/docs/locations), which is usually far closer to the user than your backend is.



## **Start the HTTP load balancer configuration**

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Network Services > Load balancing**.
* Click **Create load balancer**.
* Under **HTTP(S) Load Balancing**, click **Start configuration**.
* Select **From Internet to my VMs**, and click **Continue**.
* For **Name**, type **cdn-lb**



## **Configure the backend**

* Click **Backend configuration**.

* For **Backend services & backend buckets**, click **Create or select backend services & backend buckets > Backend buckets > Create a backend bucket**.

* For **Name**, type **cdn-bucket**

* Click **Browse** under **Cloud Storage bucket**.

* Select your bucket, and click **Select**.

* Select **Enable Cloud CDN**.

* Click **Create**.

> Yes, enabling Cloud CDN is as simple as selecting **Enable Cloud CDN**!



## **Configure the frontend**



The host and path rules determine how your traffic will be directed. For example, you could direct video traffic to one backend and image traffic to another backend. However, you are not configuring the host and path rules in this lab.

* Click **Frontend configuration**.

* Specify the following, leaving all other values with their defaults:

  | Property   | Value (type value or select option as specified) |
  | :--------- | :----------------------------------------------- |
  | Protocol   | HTTP                                             |
  | IP version | IPv4                                             |
  | IP address | Ephemeral                                        |
  | Port       | 80                                               |

* Click **Done**.



## **Review and create the HTTP load balancer**

* Click **Review and finalize**.
* Ensure that **Cloud CDN** is **enabled** for the **Backend buckets**. If it is not, return to the **backend configuration** to enable Cloud CDN.
* Click **Create**. Wait for the load balancer to be created.
* Click on the name of the load balancer (**cdn-lb**).
* Note the IP address of the load balancer. It will be referred to as `[LB_IP_ADDRESS]`.
* Open a new browser tab and navigate to **http://[LB_IP_ADDRESS]/regions.png**, replacing `[LB_IP_ADDRESS]` with the IP address of the load balancer.

> You should see an error because the load balancer needs time to be available from all edge locations. Wait a couple of minutes and keep refreshing the browser tab until you can see the image.



* When you can see the **regions.png** image (shown below), continue to the next task.

![img](https://cdn.qwiklabs.com/D4%2F8jedInLlBn2iTbKJovSxWQK1aCudDoo6W%2B%2BdPw0o%3D)



# Task 3. Verify the caching of your bucket's content



Now that you have created the HTTP load balancer for your bucket and enabled Cloud CDN, it is time to verify that the image is cached on the edge of Google's network.



## **Time the HTTP request for an image**



One way to verify that an image is cached is to time the HTTP request for the image. The first request should take significantly longer, because content is only cached at an edge location after being accessed through that location.

You already accessed and cached the **regions.png** image. Therefore, you will focus on the **cdn.png** image in this task.

* In Cloud Shell, store the IP address of the load balancer in an environment variable:

```
export LB_IP_ADDRESS=<Enter the IP address of the load balancer>
```

* Run the following command for consecutive HTTP requests:

```
for ((i=0;i<10;i++)); do curl -w  \
    "%{time_total}\n" -o /dev/null -s http://$LB_IP_ADDRESS/cdn.png; done
```

The output should look like this (**do not copy; this is example output**):

```
0.028468
0.000086
0.000102
...
```



> In this example output, the second and third request take less than 1% of the time of the first request. This demonstrates that the image was cached during the first request and accessed from an edge location on further requests. Depending on how far away you placed your storage bucket and where your closest edge location is, you will see different results.



## **Invalidate the cached content**



When an object is cached, it normally remains in the cache until it expires or is evicted to make room for new content. Sometimes, you might want to remove an object from the cache before its normal expiration time. You can force an object, or set of objects, to be ignored by the cache by requesting a cache invalidation.

* Open a new browser tab and navigate to **http://[LB_IP_ADDRESS]/cdn.png**, replacing `[LB_IP_ADDRESS]` with the IP address of the load balancer.

This image is very dated and needs to replaced with a newer image.

* In Cloud Shell, replace the old image with a newer image:

```
gsutil cp gs://cloud-training/gcpnet/cdn/updatedcdn.png gs://$YOUR_BUCKET/cdn.png
```

* Return to the browser tab for **http://[LB_IP_ADDRESS]/cdn.png**, and reload the page.

> You should see the old image because it is still cached

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Network Services > Load balancing**.
* Click on the name of the load balancer (**cdn-lb**).
* Click **Caching**.
* For the **Path pattern to invalidate**, type **/cdn.png**
* Click **Invalidate** and wait for the invalidation to complete under the **Recent cache invalidations** section.

This sends off the request to invalidate cache entries, so the command takes some time to run.

* Return to the browser tab for **http://[LB_IP_ADDRESS]/cdn.png**, and reload the page.

> You should now see a new image.



# Task 4. Review



In this lab, you configured Cloud CDN for a backend bucket by configuring an HTTP load balancer and enabling Cloud CDN with a simple checkbox. You verified the caching of the bucket's content by measuring the time it takes to access the image. The first time you accessed the image, it took longer because the cache of the edge location did not contain the image yet. All other requests were quicker because the image was provided from the cache of the edge location closest to your Cloud Shell instance.

You then replaced the image and observed that the old image was still being cached. You resolved this by invalidating the cache through the Cloud Console.

