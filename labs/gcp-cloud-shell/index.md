# Working with GCP Cloud Console and Cloud Shell

## Overview 

In this lab, you become familiar with the Google Cloud web-based interface. There are two integrated environments: a GUI (graphical user interface) environment called the Cloud Console, and a CLI (command-line interface) called Cloud Shell. In this lab, you use both environments.

Here are a few things you need to know about the Cloud Console:

- The Cloud Console is under continuous development, so occasionally the graphical layout changes. This is most often to accommodate new Google Cloud features or changes in the technology, resulting in a slightly different workflow.
- You can perform most common Google Cloud actions in the Cloud Console, but not all actions. In particular, very new technologies or sometimes detailed API or command options are not implemented (or not yet implemented) in the Cloud Console. In these cases, the command line or the API is the best alternative.
- The Cloud Console is extremely fast for some activities. The Cloud Console can perform multiple actions on your behalf that might require many CLI commands. It can also perform repetitive actions. In a few clicks you can accomplish activities that would require a lot of typing and would be susceptible to typing errors.
- The Cloud Console is able to reduce errors by offering only valid options through its menus. It can leverage access to the platform "behind the scenes" through the SDK to validate configuration before submitting changes. A command line can't do this kind of dynamic validation.



## Objectives

In this lab, you learn how to perform the following tasks:

- Get access to Google Cloud.
- Use the Cloud Console to create a Cloud Storage bucket.
- Use Cloud Shell to create a Cloud Storage bucket.
- Become familiar with Cloud Shell features.



# Task 1: Use the Cloud Console to create a bucket

In this task, you create a bucket. However, the text also helps you become familiar with how actions are presented in the lab instructions in this class and teaches you about the Cloud Console interface.



## Navigate to the Storage service and create the bucket

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Cloud Storage > Browser**.

* Click **Create bucket**.

* For **Name**, type a globally unique bucket name; leave all other values as their defaults.

* Click **Create**.

### Explore features in the Cloud Console

The Google Cloud menu contains a Notifications icon. Sometimes, feedback from the underlying commands is provided there. If you are not sure what is happening, check Notifications for additional information and history.



# Task 2: Access Cloud Shell

In this section, you explore Cloud Shell and some of its features.

You can use the Cloud Shell to manage projects and resources via command line without having to install the Cloud SDK and other tools on your computer.

Cloud shell provides the following:

- Temporary Compute Engine VM
- Command-line access to the instance via a browser
- 5 GB of persistent disk storage ($HOME dir)
- Pre-installed Cloud SDK and other tools
- gcloud: for working with Compute Engine and many Google Cloud services
- gsutil: for working with Cloud Storage
- kubectl: for working with Google Kubernetes Engine and Kubernetes
- bq: for working with BigQuery
- Language support for Java, Go, Python, Node.js, PHP, and Ruby
- Web preview functionality
- Built-in authorization for access to resources and instances

> After 1 hour of inactivity, the Cloud Shell instance is recycled. Only the /home directory persists. Any changes made to the system configuration, including environment variables, are lost between sessions.

![img](https://cdn.qwiklabs.com/%2FQ6EIl5IkKGBItOtB8tvMVGqRUTcTZCQj4gVYwTv1%2Bw%3D)

## Open Cloud Shell and explore features

In the Google Cloud menu, click **Activate Cloud Shell** (![Activate Cloud Shell](https://cdn.qwiklabs.com/ep8HmqYGdD%2FkUncAAYpV47OYoHwC8%2Bg0WK%2F8sidHquE%3D)). If prompted, click **Continue**. Cloud Shell opens at the bottom of the Cloud Console window.

There are three buttons on the far right of the Cloud Shell toolbar:

<img src="https://cdn.qwiklabs.com/xHKpKKTXWBkXrdrB1yYpaJETcizw6kdzaLuO5%2FzddbU%3D" alt="img" style="zoom:50%;" />

- **Minimize/Restore:** The first one minimizes or restores the window, giving you full access to the Cloud Console without closing Cloud Shell.
- **Open in a new window:** Having Cloud Shell at the bottom of the Cloud Console is useful when you are issuing individual commands. However, sometimes you will be editing files or want to see the full output of a command. For these uses, this button pops the Cloud Shell out into a full-sized terminal window.
- **Close terminal:** This button closes Cloud Shell. Every time you close Cloud Shell, the virtual machine is reset and all machine context is lost.

* Close Cloud Shell now.



# Task 3. Use Cloud Shell to create a Cloud Storage bucket



## Create a second bucket and verify in the Cloud Console

* Open Cloud Shell again.

* Use the gsutil command to create another bucket. Replace <BUCKET_NAME> with a globally unique name (**you can append a 2 to the globally unique bucket name you used previously**):

```
gsutil mb gs://<BUCKET_NAME>
```



* If prompted, click **Authorize**.

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Cloud Storage > Browser**, or click **Refresh** if you are already in the Storage browser. The second bucket should be displayed in the **Buckets** list.

> You have performed equivalent actions using the Cloud Console and Cloud Shell. You created a bucket using the Cloud Console and another bucket using Cloud Shell.



# Task 4: Explore more Cloud Shell features



## Upload a file 

* Open Cloud Shell.

* Click the **More** button (![More](https://cdn.qwiklabs.com/2ufrDePg5inKfodUoT2Kib4oE7II7emYn%2BypCC85FjQ%3D)) in the Cloud Shell toolbar to display further options.

* Click **Upload**. Upload any file from your local machine to the Cloud Shell VM. This file will be referred to as [MY_FILE].

<img src="https://cdn.qwiklabs.com/oJbmK6VrlyvZLPw5WdM0HJcPpuycYDtzan%2F5HL6FHjE%3D" alt="img" style="zoom:50%;" />

* In Cloud Shell, type `ls` to confirm that the file was uploaded.

* Copy the file into one of the buckets you created earlier in the lab. Replace [MY_FILE] with the file you uploaded and [BUCKET_NAME] with one of your bucket names:

```
gsutil cp [MY_FILE] gs://[BUCKET_NAME]
```

If your filename has whitespaces, be sure to place single quotes around the filename. For example, `gsutil cp ‘my file.txt' gs://[BUCKET_NAME]`

>  You have uploaded a file to the Cloud Shell VM and copied it to a bucket.



* Explore the options available in Cloud Shell by clicking on different icons in the Cloud Shell toolbar.

* Close all the Cloud Shell sessions.

# Task 5: Create a persistent state in Cloud Shell

In this section you will learn a best practice for using Cloud Shell. The gcloud command often requires you to specify values such as a **Region**, **Zone**, or **Project ID**. Entering them repeatedly increases the chance of making typing errors. If you use Cloud Shell frequently, you may want to set common values in environment variables and use them instead of typing the actual values.



## Identify available regions

* Open Cloud Shell from the Cloud Console. Note that this allocates a new VM for you.

* To list available regions, execute the following command:

```
gcloud compute regions list
```

* Select a region from the list and note the value in any text editor. This region will now be referred to as [YOUR_REGION] in the remainder of the lab.



## Create and verify an environment variable

* Create an environment variable and replace [YOUR_REGION] with the region you selected in the previous step:

```
INFRACLASS_REGION=[YOUR_REGION]
```

* Verify it with echo:

```
echo $INFRACLASS_REGION
```



You can use environment variables like this in gcloud commands to reduce the opportunities for typos and so that you won't have to remember a lot of detailed information.

> Every time you close Cloud Shell and reopen it, a new VM is allocated, and the environment variable you just set disappears. In the next steps, you create a file to set the value so that you won't have to enter the command each time Cloud Shell is reset.



## Append the environment variable to a file

* Create a subdirectory for materials used in this lab:

```
mkdir infraclass
```

* Create a file called `config` in the infraclass directory:

```
touch infraclass/config
```

* Append the value of your Region environment variable to the `config` file:

```
echo INFRACLASS_REGION=$INFRACLASS_REGION >> ~/infraclass/config
```

* Create a second environment variable for your Project ID, replacing [YOUR_PROJECT_ID] with your Project ID. You can find the project ID on the Cloud Console Home page.

```
INFRACLASS_PROJECT_ID=[YOUR_PROJECT_ID]
```

* Append the value of your Project ID environment variable to the `config` file:

```
echo INFRACLASS_PROJECT_ID=$INFRACLASS_PROJECT_ID >> ~/infraclass/config
```

* Use the source command to set the environment variables, and use the echo command to verify that the project variable was set:

```
source infraclass/config
echo $INFRACLASS_PROJECT_ID
```



> This gives you a method to create environment variables and to easily recreate them if the Cloud Shell is recycled or reset. However, you will still need to remember to issue the source command each time Cloud Shell is opened. In the next step, you modify the .profile file so that the source command is issued automatically every time a terminal to Cloud Shell is opened.

* Close and re-open Cloud Shell. Then issue the echo command again:

```
echo $INFRACLASS_PROJECT_ID
```

There will be no output because the environment variable no longer exists.



## Modify the bash profile and create persistence

```
nano .profile
```

* Add the following line to the end of the file:

```
source infraclass/config
```

* Press **Ctrl+O**, **ENTER** to save the file, and then press **Ctrl+X** to exit nano.

* Close and then re-open Cloud Shell to reset the VM.

* Use the echo command to verify that the variable is still set:

```
echo $INFRACLASS_PROJECT_ID
```

You should now see the expected value that you set in the config file.



# Task 6: Review the Google Cloud interface

Cloud Shell is an excellent interactive environment for exploring Google Cloud by using Google Cloud SDK commands like `gcloud` and `gsutil`.

You can install the Google Cloud SDK on a computer or on a VM instance in Google Cloud. The gcloud and gsutil commands can be automated by using a scripting language like bash (Linux) or Powershell (Windows). You can also explore using the command-line tools in Cloud Shell, and then use the parameters as an implementation guide in the SDK using one of the supported languages.

The Google Cloud interface consists of two parts: the Cloud Console and Cloud Shell.

The Console:

- Provides a fast way to perform tasks.
- Presents options to you, instead of requiring you to know them.
- Performs behind-the-scenes validation before submitting the commands.

Cloud Shell provides:

- Detailed control
- A complete range of options and features
- A path to automation through scripting







