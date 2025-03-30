# monolithic-deployment
This repository contains a guide for a distributed system deployment of the open source Moodle application on Amazon Web Services (AWS).

### AWS Components:
- Virtual Private Cloud (VPC)
- Subnet
- Internet Gateway
- Security Groups
- Availability Zones (AZs)

### Developer Notes:
#### Example
Example text

## Installation
Example of embedding code.

```bash
sudo apt update -y
```
### Launching an EC2 Instance
1. Open up your AWS Console and select the EC2 Service.\
2. Select "Instances" from the drop down on the left.\
3. Select "Launch instances" in the top right orange button.\
4. Provide an appropriate name for the instance in the name field, under the "Name and tags" section.\
5. Under the "OS and images" section, we are going to select a linux Ubuntu image (24.04 LTS).\
6. Under the "Instance type" section, we are going to select a "t2.micro" instancee for demonstration purposes. Details on the types of AWS instances are available here.
https://aws.amazon.com/ec2/instance-types/\

7. Under the "Key pair" section, select create a new key pair. Select "RSA" for type and ".pem" for format. Provide an appropriate name, and click create key pair. Your private keypair will automatically be downloaded to your machine.\
8. Under the "Network settings" section, Click "edit" on the corner to the right of the section heading, and enable auto-assign public IP. This will provide the EC2 instance with a publicly accessible IP address (Please not this is not a static Ip and may change when the instance is shutdown and start, or restarted. For a static IP, please look into aws elastic IPs. we'll use the default VPC and subnet for demonstration purposes, and create a new security group that allows inbound traffic on ports 80, 443, and 22.\
9. Under the "Configure storage" section, configure an 8GB GP3 EBS volume.\
10. Scroll down to the bottom of the page and select launch instance.\

### Confifuring EC2 instance
1. Remotely log into the instance via ssh using the previously downloaded key pair.
2. 2. 
