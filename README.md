# AWS Data Pipeline: Event-Driven S3 Ingestion to EFS via Serverless Lambda

**By Ember Cloud LLC**

## Lab Overview

This repository details the architecture and implementation of an event-driven data relocation pipeline on AWS. The system captures object storage events from an Amazon S3 ingress layer and automatically mounts, transfers, and persists the data downstream into an Amazon Elastic File System (EFS) network using serverless AWS Lambda execution blocks. Built as a foundational component of my validation path for the **AWS Certified Solutions Architect – Associate (SAA)** credential, this architecture implements secure multi-tier networking, private gateway endpoints, and persistent cross-service access structures.

---

## Technical Implementation Steps

### 1. Hardened Storage Tier Configuration
* Provisioned an isolated object repository (`saa-lab-5-7`) for baseline file ingestion. 
* Enforced an explicit, restrictive VPC-only bucket access policy to completely isolate the data layer from perimeter public routing.

### 2. Multi-Tier VPC Network Architecture
* Engineered a custom software-defined virtual private network (`SAA-Lab-VPC`) implementing isolated public (`SAA-Public-Subnet-A`) and private (`SAA-Private-Subnet-A`) subnet ranges.
* Configured discrete route tables, Network Access Control Lists (NACLs), and stateful Security Groups to build a defensive security boundary around internal resources.

### 3. Compute Layer & Bastion Management Planes
* Deployed a secure bastion host cluster (`SAA-Bastion`) inside the public ingress perimeter to establish an audited administrative link.
* Launched isolated backend instances (`S3-Lab-Private`) within the private subnet architecture to handle localized verification routines without open internet exposure.

### 4. Serverless Integration & Persistent File Fabrics
* Created a highly available network file system (`SAA-Lab-EFS`) configured with a localized Access Point entry root (`/lambda`).
* Deployed a serverless worker runtime (`S3ToEFSLambda`) linked to asynchronous S3 object storage creation triggers. 
* Mounted the network file structure onto the backend compute layer to confirm live, secure file relocation operations.

---

## Engineering Breakthroughs & Deep Diagnostics

* **Remediated Private Ingress Boundaries:** Fixed localized file retrieval dropouts on private compute layers by establishing a stateless **S3 VPC Gateway Endpoint**
