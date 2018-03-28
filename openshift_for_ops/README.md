# OpenShift for Ops Workshop Provisioner

This is an automated provisioner, using Ansible, for the **OpenShift for Ops**
workshop that can be found at http://redhatgov.io/workshops/openshift_for_ops/.

The provisioner currently supports deploying the workshop to AWS.

By default, this provisioner will deploy five (5) nodes per student:

- bastion
- master
- infra
- app01
- app02

## Environment Setup

### Clone Repository

Start by cloning this repository to the machine you intend to execute the
provisioner from:

```bash
git clone https://github.com/RedHatGov/redhatgov.workshops.git
```

**Change to this provisioner in the repo you cloned. The remainder of this guide
assumes you are in the `openshift_for_ops` directory of the `redhatgov.workshops`
repo.**

```bash
cd redhatgov.workshops/openshift_for_ops
```

### Install Python Packages

To use this provisioner, you will need to install a few Python packages
on the machine that you intend to execute the provisioner from.

Use `pip` to install the required packages:

```bash
pip install -r requirements.txt
```

If you are planning to use a `virtualenv` on a machine with SELinux enabled,
you will also need to copy the `selinux` Python packages to your `virtualenv`.
**If you don't know what `virtualenv` is or do not plan to use it, this can be
skipped.**

```bash
sudo dnf install -y libselinux-python

ln -s /usr/lib64/python2.7/site-packages/selinux $VIRTUAL_ENV/lib64/python2.7/site-packages/selinux
ln -s /usr/lib64/python2.7/site-packages/_selinux.so $VIRTUAL_ENV/lib64/python2.7/site-packages/_selinux.so
```

## AWS Setup

You will need to manually setup a few items in AWS and provide their information
in your [variables file](#required-variables).

### AWS Region

You will need to choose an AWS region to use. It's best to choose one that's
geographically close to where you will be delivering the workshop.

The provisioner currently only supports the following regions:

- us-east-1
- us-east-2
- us-west-1
- us-west-2

### Access Keys

Create an AWS access key and secret key if you do not already have one. Save
this information and for your [variables file](#required-variables).

https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html

### VPC

It's likely that your AWS account already has a VPC that you can use.

Go to https://console.aws.amazon.com/vpc/home and select **Subnets** in the
left column. Here you should see the available subnets in your AWS account.

You will need a subnet with enough available IPv4 addresses for your workshop.
This can be calculated using `(num_students * 5) + 1`. When you select a subnet
in the list, the **Summary** tab will show the **Available IPs**.

Take note of the **Subnet ID** (e.g. `subnet-00232f676d5697cd8`) and
**VPC ID** (e.g. `vpc-0a6c2af2159f19e1c`) for your [variables file](#required-variables).

**If you do not have a VPC**, you can create one using the [AWS Console](https://console.aws.amazon.com/vpc/home)
by clicking the **Start VPC Wizard**. Change the following options:

- **IPv4 CIDR block:** 172.35.0.0/16
- **Public subnet's IPv4 CIDR:** 172.35.0.0/16

### Route53

You will need a Route53 hosted zone to use for this provisioner. If you do not
already have a hosted zone defined, create one using the [AWS Console](https://console.aws.amazon.com/route53/home).

## Required Variables

You will need to create a variables file that contains the information specific
to your environment.

There is an example variables file located at `playbooks/vars/example.yml`. Make
a copy of this file and edit the variables to match your environment. It is
also displayed below for convenience.

```bash
cp playbooks/vars/example.yml playbooks/vars/acme_workshop.yml
```

Example variables file:

```yaml
---

aws_access_key: your_access_key
aws_secret_key: your_secret_key

ec2_region: us-east-1
ec2_vpc_id: vpc-123456789
ec2_vpc_subnet_id: subnet-987654321

route53_hosted_zone: example.com

workshop_uid: acme
num_students: 10

rhsm_username: johndoe
rhsm_password: password123
rhsm_pool: 4a5b8de7d4c4fa3fd667b223e4468ff4
```

Below are the description of each variable:

| Variable | Description |
| --- | --- |
| aws_access_key | Your AWS access key from [Access Keys](#access-keys) |
| aws_secret_key | Your AWS secret key from [Access Keys](#access-keys) |
| ec2_region | The AWS region from [AWS Region](#aws-region) |
| ec2_vpc_id | The VPC ID from [VPC](#vpc) |
| ec2_vpc_subnet_id | The VPC subnet ID from [VPC](#vpc) |
| route53_hosted_zone | The Route53 hosted zone from [Route53](#route53) |
| workshop_uid | A unique name to identify your workshop. This value will be used in the generation of your workshop hostname. For example, if `route53_hosted_zone=example.com` and `workshop_uid=acme`, the base hostname used for all instances will be `acme.example.com`. |
| num_students | The number of student environments to create. It's a good idea to create an extra one for you to use. |
| rhsm_username | Your username for RHSM (access.redhat.com) |
| rhsm_password | Your password for RHSM (access.redhat.com) |
| rhsm_pool | The RHSM pool ID that contains OpenShift repos |

## Execute Provisioner

Once you have created you variables file, you are ready to execute the
provisioner.

```bash
ansible-playbook playbooks/provision.yml -e @playbooks/vars/acme_workshop.yml
```

## Teardown Environment

Once you are done with the workshop and are ready to teardown the environment,
execute the following:

```bash
ansible-playbook playbooks/teardown.yml -e @playbooks/vars/acme_workshop.yml
```
