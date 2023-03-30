# MANUALLY-IMPLEMENTING-AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOG

## Table of Contents

Introduction
Starting Off Your AWS Cloud Project
SET UP A VIRTUAL PRIVATE NETWORK (VPC)
Creating the Subnets
Creating the Route Tables
Creating an Elastic IP
Creating a NAT Gateway
Creating the Security Groups
Assigning Certificate and creating Hosted Zone
Creating the Elastic File System (EFS)
Creating the Relational Database Service (RDS), Key Management Service (KMS) and Subnet Groups
Creating our Resources
Bastion AMI installation
Nginx AMI installation
Webserver AMI installation
Creating our AMI images
Creating our Target groups
Creating our Load Balancers
Creating the Autoscaling Groups
Creating the Route53 records

## Introduction

In this project, you will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (Choose an interesting name for it) that uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

![infrastructure-architecture](/Images/resource-architecture.png)
