# Ansible role to create dummy files on Amazon Elastic File System (EFS) mounts

## What is it?

Amazon EFS uses a credit system to determine when file systems can burst. Each file system earns credits over time at a baseline rate that is determined by the size of the file system, and uses credits whenever it reads or writes data. The baseline rate is 50 MiB/s per TiB of storage (equivalently, 50 KiB/s per GiB of storage).

Accumulated burst credits give the file system permission to drive throughput above its baseline rate. A file system can drive throughput continuously at its baseline rate, and whenever it's inactive or driving throughput below its baseline rate, the file system accumulates burst credits. However, if the burst credit become depleted the user can't drive throughput into the file system. To avoid this happening you can create dummy files to increase the EFS file system size and gain more burst credits.

This ansible role can be used to automatically create dummy files on EFS file systems. The playbook uses the ec2 module to launch a new instance, and run into the user_data a shell script to mount the EFS file system and create some dummy files using the dd command. After the shell script complete the instance will be terminate.

Notice that before executing the playbook you need to change some variables under the file vars/main.yml to match your environment configuration.

## Prerequisites

This ansible role was tested with ansible version 2.4.0.0 only.

To run the playbook you need at least the below IAM policies to be able to launch a new instance:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:RunInstances",
      "Resource": [
        "arn:aws:ec2:region::image/ami-acd005d5",
        "arn:aws:ec2:region:account:instance/*",
        "arn:aws:ec2:region:account:volume/*",
        "arn:aws:ec2:region:account:key-pair/*",
        "arn:aws:ec2:region:account:security-group/*",
        "arn:aws:ec2:region:account:subnet/*",
        "arn:aws:ec2:region:account:network-interface/*"
      ]
    }
  ]
}
```

If you need more information about IAM policies, please refer to [Example Policies for Working With the AWS CLI or an AWS SDK](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ExamplePolicies_EC2.html#iam-example-runinstances).

## Usage

Install this ansible role using galaxy:

```sh
$ ansible-galaxy install diesant.aws_efs_burst_credits
```

Before executing the ansible playbook, you first need to change some variables under the file vars/main.yml to match your environment configuration.

## Role Variables

```yaml
vars:

# If you are not using an IAM role, configure your AWS Credentials and the Region
aws_access_key: "AKIAIOSFODNN7EXAMPLE"
aws_secret_key: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
aws_region: "eu-west-1"

# The instance type that will be used to run the script that creates the dummy files
instance_types: ["c4.large"]

# The AMI id that will be used to launch the instance
ami_id: "ami-ebd02392"

# The number of instances to be launched
count_number: "1"

# Launch the instance with a public IP address or not
yon: "yes"

# The instance tag Name
ec2_tag_Name: "aws_efs_burst_credits"

# The EFS file system id that will be mounted and where the dummy files will be created
efs_filesystem: "file-system-id.efs.aws-region.amazonaws.com:/"

# The VPC subnet where the instance will be launched
subnet: "subnet-650b1e13"

# The security group that will be attached to the instance that will be launched
security_group: "sg-b09517c8"

# The number of dummy files to be created
dummy_files: "10"
```

## Running the playbook

After changing the variables you're good to run the playbook:

```sh
$ ansible-playbook -i inventory role.yml
```
