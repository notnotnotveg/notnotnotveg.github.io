---
title: "RCE into EA using Kubeflow Notebooks"
date: 2024-04-03T13:00:00-01:00
categories:
  - blog
tags:
  - Responsible Disclosure
toc: True
toc_label: Responsible Disclosure
toc_icon: "bug"
---

### Intro
Multiple articles and blog posts written about [Kubeflow](https://www.kubeflow.org/) intrigued me to dive (albeit shallow) into and scour the depths of Shodan one evening to see what I could find. With not too many of such exposed instances to begin with, and accounting for what little the apex predators left behind ([Microsoft's blog post](https://www.microsoft.com/en-us/security/blog/2020/06/10/misconfigured-kubeflow-workloads-are-a-security-risk/)), I stumbled across one such Kubeflow Dashboard, with a misconfiguration, that got me Remote Code Execution (RCE) into **Electronic Arts (EA)**.

### What is Kubeflow
Directly quoting the Kubeflow Project home page :


> *The Kubeflow project is dedicated to making deployments of machine learning (ML) workflows on Kubernetes simple, portable and scalable. Our goal is not to recreate other services, but to provide a straightforward way to deploy best-of-breed open-source systems for ML to diverse infrastructures.*


One component of Kubeflow is the ability to run Kubeflow Notebooks, including _Jupyter notebooks_. 

### Enumeration
Kubeflow dashboards accessible to the public Internet can be identified on Shodan using the query :
```
http.title:"Kubeflow Central Dashboard"
```

### Exploitation 
Most Kubeflow Dashboards use authentication to protect exposed instances from being exploited using Jupyter Notebooks. MLSecOps has a great [blog post](https://mlsecops.com/resources/hacking-ai-account-hijacking-and-internal-network-attacks-in-kubeflow) about stealing this token using XSS and using it to access Jupyter notebooks. 

I happened to find a Kubeflow Dashboard with the Shodan query above, hosted on a subdomain of `ea.com`. This Kubeflow deployment did not allow creation of new Notebooks. However, as I browsed through the different namespaces, I observed Notebooks that were already created for a specific `namespace`.

![kubeflow1 1](https://github.com/notnotnotveg/notnotnotveg.github.io/assets/65092714/5ed5a3aa-e711-442f-b926-732fa0e220c6)

Looking into this notebook, an existing [Terminal](https://jupyterlab.readthedocs.io/en/latest/user/terminal.html) was found to be running :
![kubeflow2](https://github.com/notnotnotveg/notnotnotveg.github.io/assets/65092714/4142323b-5dbc-4ca7-afa6-77faca3f0b9a)

Accessing this Terminal provided a bash prompt on a container running in this namespace. Leveraging this shell, the following impact was proved :
#### 1. Access to Instance Metadata services 
IAM credentials for the AWS account were enumerated after accessing AWS IMDSv1  :

![5DC51F77-85E4-4377-9EDD-D6B36D8F4374](https://github.com/notnotnotveg/notnotnotveg.github.io/assets/65092714/bea4e90e-1415-4863-ac21-95ddcd23f6e1)

#### 2. Access to Internal (RFC-1918) Endpoints
Using public DNS dumps an internal endpoint was identified and found to be accessible in our shell. 

As an example, the following subdomain resolves to an RFC-1918 IP address on the public Internet :

```
$ dig +short @8.8.8.8 [REDACTED].ea.com
internal-prd-gitlab-elb[REDACTED].us-east-1.elb.amazonaws.com.
10.28.[REDACTED]
10.28.[REDACTED]
```

This endpoint, which is a Gitlab instance, was found to be accessible over the network from this container :
![kubeflow4](https://github.com/notnotnotveg/notnotnotveg.github.io/assets/65092714/75797c1a-6999-4e31-8af4-75cef745ffc3)

This is the point where we conclude any further testing on this specific instance, since access to the infrastructure and the ability to escalate privileges in the environment were demonstrated.

### Disclosure Timeline 

| <!-- -->Date   | <!-- -->Action                                                          |
| -------------- | ----------------------------------------------------------------------- |
| March 02, 2024 | Submitted details of the issue to the EA Product Security Response Team |
| March 04, 2024 | Report Acknowledged by EA.                                              |
| March 18, 2024 | Draft of Disclosure report sent to EA.                                  |
| March 20, 2024 | Issue Confirmed to be remediated by EA.                                 |
| March 29, 2024 | Draft of Disclosure report reviewed and approved by EA.                 |
| April 03, 2024 | Report disclosed.                                                       |

### Acknowledgements 
A shout out to the EA Product Security Response Team for being prompt on their communications, and making the process for disclosing the issue as seamless and comfortable as it can be.
