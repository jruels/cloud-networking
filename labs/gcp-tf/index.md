# Automating the Deployment of Networks Using Terraform



## Overview 

Terraform enables you to safely and predictably create, change, and improve infrastructure. It is an open-source tool that codifies APIs into declarative configuration files that can be shared among team members, treated as code, edited, reviewed, and versioned.

In this lab, you create a Terraform configuration with a module to automate the deployment of a custom network with resources. Specifically, you deploy 3 networks with firewall rules and VM instances, as shown in this network diagram:

![img](https://cdn.qwiklabs.com/b8MOrWyQ%2BNvgPrCkNn%2BPdX1j2Rik1qrrpzRwfHCoHpg%3D)

### Objectives

In this lab, you learn how to perform the following tasks:

- Create a configuration for a custom-mode network
- Create a configuration for a firewall rule
- Create a module for VM instances
- Create a configuration for an auto-mode network
- Create and deploy a configuration
- Verify the deployment of a configuration



# Task 1. Set up Terraform and Cloud Shell

Configure your Cloud Shell environment to use Terraform.



## Install Terraform

Terraform is now integrated into Cloud Sell. Verify which version is installed.

* In the Cloud Console, click **Activate Cloud Shell** (![Cloud Shell](https://cdn.qwiklabs.com/ep8HmqYGdD%2FkUncAAYpV47OYoHwC8%2Bg0WK%2F8sidHquE%3D)).
* If prompted, click **Continue**.
* Confirm that Terraform is installed by running the following command:

```bash
terraform --version
```

The output should look like this (**do not copy; this is example output**):

```
Terraform v0.12.24
```

> Don't worry if you get a warning that the version of Terraform is out of date, as the lab instructions will work with Terraform v0.12.9 and later. 

* Create a directory for your Terraform configuration by running the following command:

```bash
mkdir tfnet
```

* In Cloud Shell, click **Open editor** (![Cloud Shell Editor](https://cdn.qwiklabs.com/zxK8nW520maN72Qq6D1Lt9gCeDh7QOMGWCwhny5S8sQ%3D)).

* Expand the **tfnet** folder in the left pane of the code editor.



## Initialize Terraform

Terraform uses a plugin-based architecture to support the numerous infrastructure and service providers available. Each "provider" is its own encapsulated binary distributed separately from Terraform itself. Initialize Terraform by setting Google as the provider.

* To create a new file, click **File** > **New File**.
* Name the new file **provider.tf**, and then open it.
* Copy the code into provider.tf:

```
provider "google" {}
```



* Initialize Terraform by running the following command:

```bash
cd tfnet
terraform init
```

The output should look like this (**do not copy; this is example output**):

```
* provider.google: version = "~> 3.36"
Terraform has been successfully initialized!
```

You are now ready to work with Terraform in Cloud Shell.



# Task 2. Create managementnet and its resources

Create the custom-mode network **managementnet** along with its firewall rule and VM instance (**managementnet-us-vm**).



## Configure managementnet

Create a new configuration and define **managementnet**.

* To create a new file, click **File** > **New File**.
* Name the new file **managementnet.tf**, and then open it.
* Copy the following base code into managementnet.tf:

```
# Create the managementnet network
resource [RESOURCE_TYPE] "managementnet" {
name = [RESOURCE_NAME]
#RESOURCE properties go here
}
```



This base template is a great starting point for any Google Cloud resource. The **name** field allows you to name the resource, and the **type** field allows you to specify the Google Cloud resource that you want to create. You can also define properties, but these are optional for some resources.

* In managementnet.tf, replace `[RESOURCE_TYPE]` with `"google_compute_network"`

The **google_compute_network** resource is a VPC network. 

* In managementnet.tf, replace `[RESOURCE_NAME]` with `"managementnet"`

* Add the following property to managementnet.tf:

```
auto_create_subnetworks = "false"
```

Unlike an auto-mode network, a custom mode network does not automatically create a subnetwork in each region. Therefore, you are setting **auto_create_subnetworks** to `false`.

* Verify that managementnet.tf looks like this:

```
 # Create the managementnet network
 resource "google_compute_network" "managementnet" {
 name = "managementnet"
 auto_create_subnetworks = "false"
 }
```

* To save managementnet.tf, click **File** > **Save**.



## Add a subnet to managementnet

Add **managementsubnet-us** to the VPC network.

* Add the following resource to managementnet.tf:

```
# Create managementsubnet-us subnetwork
resource "google_compute_subnetwork" "managementsubnet-us" {
name          = "managementsubnet-us"
region        = "us-central1"
network       = google_compute_network.managementnet.self_link
ip_cidr_range = "10.130.0.0/20"
}
```

> The **google_compute_subnetwork** resource is a subnet. You are specifying the name, region, VPC network and IP CIDR range for **managementsubnet-us** .

* To save managementnet.tf, click **File** > **Save**.



## Configure the firewall rule

Define a firewall rule to allow HTTP, SSH, and RDP traffic on managementnet

* Add the following base code to managementnet.tf:

```
# Add a firewall rule to allow HTTP, SSH, and RDP traffic on managementnet
resource [RESOURCE_TYPE] "managementnet-allow-http-ssh-rdp-icmp" {
name = [RESOURCE_NAME]
#RESOURCE properties go here
}
```

* In managementnet.tf, replace `[RESOURCE_TYPE]` with `"google_compute_firewall"`

> The **google_compute_firewall** resource is a firewall rule. 

* In managementnet.tf, replace `[RESOURCE_NAME]` with `"managementnet-allow-http-ssh-rdp-icmp"`

* Add the following property to managementnet.tf:

```
network = google_compute_network.managementnet.self_link
```

> Because this firewall rule depends on its network, you are using the **google_compute_network.managementnet.self_link** reference to instruct Terraform to resolve these resources in a dependent order. In this case, the network is created before the firewall rule.

* Add the following properties to managementnet.tf:

```
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
source_ranges = ["0.0.0.0/0"]
allow {
    protocol = "icmp"
    }
```

The list of **allow** rules specify which protocols and ports are permitted.

* Verify that your additions to managementnet.tf look like this:

```
# Add a firewall rule to allow HTTP, SSH, and RDP traffic on managementnet
resource "google_compute_firewall" "managementnet-allow-http-ssh-rdp-icmp" {
name = "managementnet-allow-http-ssh-rdp-icmp"
network = google_compute_network.managementnet.self_link
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
source_ranges = ["0.0.0.0/0"]
allow {
    protocol = "icmp"
    }
}
```

* To save managementnet.tf, click **File** > **Save**.

## Configure the VM instance

Define the VM instance by creating a VM instance module. A module is a reusable configuration inside a folder. You will use this module for all VM instances of this lab.

* To create a new folder inside **tfnet**, select the **tfnet** folder, and then click **File** > **New Folder**.
* Name the new folder **instance**.
* To create a new file inside **instance**, select the **instance** folder, and then click **File** > **New File**.
* Name the new file **main.tf**, and then open it.

You should have the following folder structure in Cloud Shell:

![img](https://cdn.qwiklabs.com/TFj5%2FzU4WzR%2FVlqolKgzVRmkO91s5%2Fco7NEO4ydDYZQ%3D)

* Copy the following base code into **main.tf**:

```
resource [RESOURCE_TYPE] "vm_instance" {
  name = [RESOURCE_NAME]
  #RESOURCE properties go here
}
```



* In main.tf, replace `[RESOURCE_TYPE]` with `"google_compute_instance"`

> The **google_compute_instance** resource is a Compute Engine instance.

* In main.tf, replace `[RESOURCE_NAME]` with `"${var.instance_name}"`

Because you will be using this module for all VM instances, you are defining the instance name as an input variable. This allows you to control the name of the variable from managementnet.tf.

* Add the following properties to main.tf:

```
  zone         = "${var.instance_zone}"
  machine_type = "${var.instance_type}"
```

These properties define the zone and machine type of the instance as input variables.

* Add the following properties to main.tf:

```
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
      }
  }
```

This property defines the boot disk to use the Debian 9 OS image. Because all four VM instances will use the same image, you can hard-code this property in the module.

* Add the following properties to main.tf:

```
  network_interface {
    subnetwork = "${var.instance_subnetwork}"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
```

This property defines the network interface by providing the subnetwork name as an input variable and the access configuration. Leaving the access configuration empty results in an ephemeral external IP address (required in this lab). To create instances with only an internal IP address, remove the access_config section.

* Define the 4 input variables at the top of main.tf and verify that main.tf looks like this, including brackets `{}`:

```
variable "instance_name" {}
variable "instance_zone" {}
variable "instance_type" {
  default = "n1-standard-1"
  }
variable "instance_subnetwork" {}
resource "google_compute_instance" "vm_instance" {
  name         = "${var.instance_name}"
  zone         = "${var.instance_zone}"
  machine_type = "${var.instance_type}"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
      }
  }
  network_interface {
    subnetwork = "${var.instance_subnetwork}"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
}
```

By giving **instance_type** a default value, the variable is optional. The **instance_name**, **instance_zone**, and **instance_subnetwork** are required, and you will define them in managementnet.tf.

* To save main.tf, click **File** > **Save**.

* Add the following VM instance to managementnet.tf:

```
# Add the managementnet-us-vm instance
module "managementnet-us-vm" {
  source           = "./instance"
  instance_name    = "managementnet-us-vm"
  instance_zone    = "us-central1-a"
  instance_subnetwork = google_compute_subnetwork.managementsubnet-us.self_link
}
```

This resource is leveraging the module in the **instance** folder and provides the name, zone, and network as inputs. Because this instance depends on a VPC network, you are using the **google_compute_network.managementsubnet-us.self_link** reference to instruct Terraform to resolve these resources in a dependent order. In this case, the subnet is created before the instance.

> The benefit of writing a Terraform module is that it can be reused across many configurations. Instead of writing your own module, you can also leverage existing modules from the [Terraform Module registry](https://registry.terraform.io/browse?provider=google&verified=true).

* To save managementnet.tf, click **File** > **Save**.

## Create managementnet and its resources

It's time to apply the managementnet configuration.

* Rewrite the Terraform configurations files to a canonical format and style by running the following command:

```bash
terraform fmt
```

> If you get an error, revisit the previous steps to ensure that your configuration matches the lab instructions.

* Initialize Terraform by running the following command:

```bash
terraform init
```

The output should look like this (**do not copy; this is example output**):

```
Initializing modules...
- managementnet-us-vm in instance
...
* provider.google: version = "~> 3.36"
Terraform has been successfully initialized!
```

> If you get an error, revisit the previous steps to ensure that you have the correct folder/file structure. 

* Create an execution plan by running the following command:

```
terraform plan
```

The output should look like this (**do not copy; this is example output**):

```
...
Plan: 4 to add, 0 to change, 0 to destroy.
...
```



Terraform determined that the following 4 resources need to be added:

| Name                                  | Description                                    |
| :------------------------------------ | :--------------------------------------------- |
| managementnet                         | VPC network                                    |
| managementsubnet-us                   | Subnet of managementnet in us-central1         |
| managementnet_allow_http_ssh_rdp_icmp | Firewall rule to allow HTTP, SSH, RDP and ICMP |
| managementnet-us-vm                   | VM instance in us-central1-a                   |

* Apply the desired changes by running the following command:

```
terraform apply
```



* Confirm the planned actions by typing:

```
yes
```

The output should look like this (**do not copy; this is example output**):

```
...
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```

Notice that when the VPC network is created, the firewall rule and subnet are created. After the subnet is created, the VM instance is created. That's because the firewall rule and subnet relied on the network, and the VM instance relied on the subnet through `self_link` references.

> If you get an error during the execution, revisit the previous steps to ensure that you have the correct folder/file structure. 



## Verify managementnet and its resources

In the Cloud Console, verify that the resources were created.

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **VPC network** > **VPC networks**.
* View the **managementnet** VPC network with its subnetwork.
* On the **Navigation menu**, click **VPC network** > **Firewall**.
* View the **managementnet-allow-http-ssh-rdp-icmp** firewall rule for the VPC network that was created.
* On the **Navigation menu**, click **Compute Engine** > **VM instances**.
* Note the **managementnet-us-vm** instance.
* Return to Cloud Shell.



# Task 3. Create privatenet and its resources

Create the custom-mode network **privatenet** along with its firewall rule and VM instance (**privatenet-us-vm**).



## Configure privatenet

Create a new configuration and define **privatenet**.

* To create a new file, click **File** > **New File**.
* Name the new file **privatenet.tf**, and then open it.

You should have the following folder structure in Cloud Shell:

![privatenet_folder.png](https://cdn.qwiklabs.com/Iqn2nCK5pdbnZfIPjEe0W15v8Ue2F5tMRTGzopfDQzg%3D)

* Add the VPC network by copying the following code into privatenet.tf:

```
# Create privatenet network
resource "google_compute_network" "privatenet" {
name                    = "privatenet"
auto_create_subnetworks = false
}
```

* Add the `privatesubnet-us` subnet resource to privatenet.tf:

```
# Create privatesubnet-us subnetwork
resource "google_compute_subnetwork" "privatesubnet-us" {
name          = "privatesubnet-us"
region        = "us-central1"
network       = google_compute_network.privatenet.self_link
ip_cidr_range = "172.16.0.0/24"
}
```



* Add the `privatesubnet-eu` subnet resource to privatenet.tf:

```
# Create privatesubnet-eu subnetwork
resource "google_compute_subnetwork" "privatesubnet-eu" {
name          = "privatesubnet-eu"
region        = "europe-west1"
network       = google_compute_network.privatenet.self_link
ip_cidr_range = "172.20.0.0/24"
}
```



* To save privatenet.tf, click **File** > **Save**.

## Configure the firewall rule

Define a firewall rule to allow HTTP, SSH, and RDP traffic on privatenet.

* Add the firewall resource to privatenet.tf:

```
# Create a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on privatenet
resource "google_compute_firewall" "privatenet-allow-http-ssh-rdp-icmp" {
name    = "privatenet-allow-http-ssh-rdp-icmp"
network = google_compute_network.privatenet.self_link
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
source_ranges = ["0.0.0.0/0"]
allow {
    protocol = "icmp"
    }
}
```



> Alternatively, you could create a module for the firewall rule because the only difference to the previous firewall rule is the VPC network that it applies to.

* To save privatenet.tf, click **File** > **Save**.



## Configure the VM instance

Use the instance module to configure **privatenet-us-vm**.

* Add the VM instance resource to privatenet.tf:

```
# Add the privatenet-us-vm instance
module "privatenet-us-vm" {
  source           = "./instance"
  instance_name    = "privatenet-us-vm"
  instance_zone    = "us-central1-a"
  instance_subnetwork = google_compute_subnetwork.privatesubnet-us.self_link
}
```

* To save privatenet.tf, click **File** > **Save**.



## Create privatenet and its resources

It's time to apply the privatenet configuration.

* Rewrite the Terraform configurations files to a canonical format and style by running the following command:

```
terraform fmt
```



> If you get an error, revisit the previous steps to ensure that your configuration matches the lab instructions. 

* Initialize Terraform by running the following command:

```
terraform init
```

The output should look like this (**do not copy; this is example output**):

```
Initializing modules...
- privatenet-us-vm in instance
...
* provider.google: version = "~> 3.36"
Terraform has been successfully initialized!
```

> If you get an error, revisit the previous steps to ensure that you have the correct folder/file structure. 

* Create an execution plan by running the following command:

```
terraform plan
```



The output should look like this (**do not copy; this is example output**):

```
...
Plan: 5 to add, 0 to change, 0 to destroy.
...
```



Terraform determined that the following 5 resources need to be added:

| Name                               | Description                                    |
| :--------------------------------- | :--------------------------------------------- |
| privatenet                         | VPC network                                    |
| privatesubnet-us                   | Subnet of privatenet in us-central1            |
| privatesubnet-eu                   | Subnet of privatenet in europe-west1           |
| privatenet-allow-http-ssh-rdp-icmp | Firewall rule to allow HTTP, SSH, RDP and ICMP |
| privatenet-us-vm                   | VM instance in us-central1-a                   |

* Apply the desired changes by running the following command:

```
terraform apply
```



* Confirm the planned actions by typing:

```
yes
```



The output should look like this (**do not copy; this is example output**):

```
...
Apply complete! Resources: 5 added, 0 changed, 0 destroyed.
```



> If you get an error during the execution, revisit the previous steps to ensure that you have the correct folder/file structure. 



## Verify privatenet and its resources

In the Cloud Console, verify that the resources were created.

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **VPC network** > **VPC networks**.
* View the **privatenet** VPC network with its subnetworks.
* On the **Navigation menu**, click **VPC network** > **Firewall**.
* View the **privatenet-allow-http-ssh-rdp-icmp** firewall rule for the VPC network that was created.
* On the **Navigation menu**, click **Compute Engine** > **VM instances**.
* Note the internal IP addresses for **privatenet-us-vm**.
* For **managementnet-us-vm**, click **SSH** to launch a terminal and connect.
* To test connectivity to **privatenet-us-vm**'s internal IP address, run the following command in the SSH terminal (replacing privatenet-us-vm's internal IP address with the value noted earlier):

```bash
ping -c 3 <Enter privatenet-us-vm's internal IP here>
```

> This should not work because both VM instances are on separate VPC networks!

* Return to Cloud Shell.



# Task 4. Create mynetwork and its resources

Create the auto-mode network **mynetwork** along with its firewall rule and two VM instances (**mynet-us-vm** and **mynet-eu-vm**).



## Configure mynetwork

* Create a new configuration and define **mynetwork**.

  1. To create a new file, click **File** > **New File**.
  2. Name the new file **mynetwork.tf**, and then open it.

  You should have the following folder structure in Cloud Shell:

  ![mynetwork_folder.png](https://cdn.qwiklabs.com/FW5jqqwpXDMiLy7wt9SlNmQRYA%2BNQJ98%2FbXh21U%2FfL8%3D)

  

* Copy the following code into mynetwork.tf:

```
# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
name                    = "mynetwork"
#RESOURCE properties go here
}
```



* Add the following property to mynetwork.tf:

```
auto_create_subnetworks = "true"
```



By definition, an auto-mode network automatically creates a subnetwork in each region. Therefore, you are setting **auto_create_subnetworks** to **true**.

* Verify that mynetwork.tf looks like this:

```
# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
name                    = "mynetwork"
auto_create_subnetworks = true
}
```



* To save mynetwork.tf, click **File** > **Save**.



## Configure the firewall rule

Define a firewall rule to allow HTTP, SSH, and RDP traffic on mynetwork.

* Add the firewall resource to mynetwork.tf:

```
# Create a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource "google_compute_firewall" "mynetwork_allow_http_ssh_rdp_icmp" {
name    = "mynetwork-allow-http-ssh-rdp-icmp"
network = google_compute_network.mynetwork.self_link
allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
    }
source_ranges = ["0.0.0.0/0"]
allow {
    protocol = "icmp"
    }
}
```



* To save mynetwork.tf, click **File** > **Save**.



## Configure the VM instance

Use the instance module to configure **mynetwork-us-vm** and **mynetwork-eu-vm**.

* Add the following VM instances to mynetwork.tf:

```
# Create the mynet-us-vm instance
module "mynet-us-vm" {
  source           = "./instance"
  instance_name    = "mynet-us-vm"
  instance_zone    = "us-central1-a"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}
# Create the mynet-eu-vm" instance
module "mynet-eu-vm" {
  source           = "./instance"
  instance_name    = "mynet-eu-vm"
  instance_zone    = "europe-west1-d"
  instance_subnetwork = google_compute_network.mynetwork.self_link
}
```

* To save mynetwork.tf, click **File** > **Save**.



## Create mynetwork and its resources

It's time to apply the mynetwork configuration.

* Rewrite the Terraform configurations files to a canonical format and style by running the following command:

```
terraform fmt
```

> If you get an error, revisit the previous steps to ensure that your configuration matches the lab instructions.



* Initialize Terraform by running the following command:

```
terraform init
```

The output should look like this (**do not copy; this is example output**):

```
Initializing modules...
- mynet-eu-vm in instance
- mynet-us-vm in instance
...
* provider.google: version = "~> 3.36"
Terraform has been successfully initialized!
```



> If you get an error, revisit the previous steps to ensure that you have the correct folder/file structure. 

* Create an execution plan by running the following command:

```
terraform plan
```



The output should look like this (**do not copy; this is example output**):

```
...
Plan: 4 to add, 0 to change, 0 to destroy.
...
```



Terraform determined that the following 4 resources need to be added:

| Name                              | Description                                    |
| :-------------------------------- | :--------------------------------------------- |
| mynetwork                         | VPC network                                    |
| mynetwork-allow-http-ssh-rdp-icmp | Firewall rule to allow HTTP, SSH, RDP and ICMP |
| mynet-us-vm                       | VM instance in us-central1-a                   |
| mynet-eu-vm                       | VM instance in europe-west1-d                  |

* Apply the desired changes by running the following command:

```
terraform apply
```



* Confirm the planned actions by typing:

```
yes
```



The output should look like this (**do not copy; this is example output**):

```
...
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```



> If you get an error during the execution, revisit the previous steps to ensure that you have the correct folder/file structure. 



## Verify mynetwork and its resources

In the Cloud Console, verify that the resources were created.

* In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **VPC network** > **VPC networks**.
* View the **mynetwork** VPC network with its subnetworks.
* On the **Navigation menu**, click **VPC network** > **Firewall**.
* View the **mynetwork-allow-http-ssh-rdp-icmp** firewall rule for the VPC network that was created.
* On the **Navigation menu**, click **Compute Engine** > **VM instances**.
* View the **mynet-us-vm** and **mynet-eu-vm** instances.
* Note the internal IP addresses for **mynet-eu-vm**.
* For **mynet-us-vm**, click **SSH** to launch a terminal and connect.
* To test connectivity to **mynet-eu-vm**'s internal IP address, run the following command in the SSH terminal (replacing mynet-eu-vm's internal IP address with the value noted earlier):

```bash
ping -c 3 <Enter mynet-eu-vm's internal IP here>
```



> This should work because both VM instances are on the same network, and ICMP traffic is allowed!



# Task 5. Review

In this lab, you created Terraform configurations and modules to automate the deployment of a custom network. As the configuration changes, Terraform can determine what changed and create incremental execution plans, which allows you to build your overall configuration step-by-step.

The instance module allowed you to re-use the same resource configuration for multiple resources while providing properties as input variables. You can leverage the configurations and modules that you created as a starting point for future deployments.



# Task6. Cleanup 

In the `tfnet` directory run `terraform destroy`

* Confirm the planned actions by typing:

```
yes
```

The output should look like this (**do not copy; this is example output**):

```
...
Destroy complete! Resources: 0 added, 0 changed, 15 destroyed.
```

