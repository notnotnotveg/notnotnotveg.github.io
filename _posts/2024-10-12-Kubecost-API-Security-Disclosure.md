---
title: "The Cost of Exposing your Kubecost"
date: 2024-10-12T13:00:00-01:00
categories:
  - blog
tags:
  - Responsible Disclosure
toc: True
toc_label: Responsible Disclosure
toc_icon: "bug"
---

### Intro :
During a recent engagement of reviewing a Kubernetes environment I came across a service called "Kubecost", which is intended to be used for monitoring a Kubernetes cluster and provide recommendations to reduce the cost associated with running it. 

Given this basic understanding of the functionality, I expected it to be a service that would not tend to provide any data or configurations it collects from the Kubernetes cluster it monitors (possibly by scraping the associated Kubernetes API endpoint). I did not find any such documented APIs or endpoints in the documentation that would indicate the same.

After exploring almost all the functionalities of the Web UI, I see a call to an interesting endpoint in my BurpSuite proxy logs. This endpoint returned back detailed information of all the Pods in the cluster ! The response was identical to what `/api/v1/pods` on the Kubernetes API provides. Essentially, this endpoint revealed the detailed configuration for Kubernetes resources to anyone who can access it over the network.  


### What is Kubecost

Directly quoting the Kubecost documenation:

> Kubecost is a monitoring application which provides real-time cost visibility and insights for teams using Kubernetes, helping you continuously reduce your cloud costs.



### Enumeration 

Kubecost dashboards accessible to the public Internet can be identified on (Shodan)[https://www.shodan.io/] using the query:

```
http.title:"Kubecost"
http.favicon.hash:611531125
```

A [nuclei](https://docs.projectdiscovery.io/tools/nuclei) template to identify unauthenticated Kubecost dasboards can be found here: 

[unauth-kubecost](https://github.com/projectdiscovery/nuclei-templates/blob/main/http/misconfiguration/unauth-kubecost.yaml).

### Exploitation 
Kubecost is recommended to be run inside trusted networks, since the service does not support authentication and authorization on the dashboard by default. However, in the case that such a Kubecost dashboard is accessible over the network, the following undocumented endpoints (at the time this was reported) are accessible: 
```
/model/allNodes
/model/allPods
/model/allDeployments  
/model/allDaemonSets
```

Though the configurations and information returned are inherently not sensitive, it may leak sensitive information about the infrastructure, and Kubernetes clusters that are misconfigured may leak secrets. As an example, secrets that may be configured as environment variables or arguments, compared to configuring them as Kubernetes Secrets, can expose sensitive data.

A simple process to manually check for such secrets : 
1. Fetch the Pod Configuration from a Kubecost dashboard that is exposed: 
```bash
curl -s "https://[KUBECOST-HOST]/model/allPods" > allPods
```
2. Search for secrets in the response using a tool such as [gron](https://github.com/tomnomnom/gron): 
```bash
gron allPods | grep  -i "secret\|password"
```


### Recommendations to users 

Users who deploy Kubecost are urged to review where their Kubecost dashboard is accessible from, and who it is accessible to. As a vast majority of these dashboards reviewed as part of this research were found to be misconfigured, a multiple step approach is recommended for defending against malicious access.

The most effective way for users to protect their clusters is by deploying the dashboard in a trusted environment, that is only accessible to the administrators of the Kubernetes cluster. 

Adding authentication and authorization to the deployment is essential for cases where lateral movement within a network is a risk. The Kubecost team highlights the following as resources for the same: 

[https://docs.kubecost.com/install-and-configure/install/ingress-examples#basic-auth-example](https://docs.kubecost.com/install-and-configure/install/ingress-examples#basic-auth-example)  
[https://docs.kubecost.com/install-and-configure/advanced-configuration/user-management-oidc](https://docs.kubecost.com/install-and-configure/advanced-configuration/user-management-oidc)  
[https://docs.kubecost.com/install-and-configure/advanced-configuration/user-management-saml](https://docs.kubecost.com/install-and-configure/advanced-configuration/user-management-saml)

In addition, as a step for defense in depth, it is highly recommended that administrators leverage Kubernetes Secrets to load secrets into pods, instead of adding them as environment variables or arguments.

![Pasted image 20241006184229](https://github.com/user-attachments/assets/909257f0-db30-492b-b683-48f67b0d37cd)

As part of the fix after disclosure, Kubecost has updated the [deployment guide](https://docs.kubecost.com/install-and-configure/install/first-time-user-guide) to include the following. 

The [API documentation](https://docs.kubecost.com/apis/diagnostic-apis/api-costmodel-cache) has also been updated to list all the endpoints that may provide information about the different resources of the Kubernetes Cluster.  

![Pasted image 20241012155106](https://github.com/user-attachments/assets/1a3393fb-4872-4b47-8ca2-711c9e879e9a)

### Disclosure Timeline
The standard Vulnerability Disclosure Process for Responsible Pubic Disclosure was followed, giving the team 90 days to address the issue.

| <!-- -->Date     | <!-- -->Action                                                                                                        |
| ---------------- | --------------------------------------------------------------------------------------------------------------------- |
| May 21, 2024     | Submitted report of the issue to the Kubecost Vulnerability Disclosure Contact                                        |
| May 22, 2024     | Response : Issue is expected behavior.                                                                                |
| May 22, 2024     | Details of Impact provided to Kubecost Team (First Attempt)                                                           |
| Jun 23, 2024     | Details of Impact provided to Kubecost Team (Second Attempt)                                                          |
| Aug 2, 2024      | Details of Impact provided to Kubecost Team (Third Attempt)                                                           |
| Aug 3, 2024      | Guidance for remediation (beyond deployment guidance) requested by Kubecost Team.                                     |
| Aug 4, 2024      | Details of undocumented APIs and information they leak provided to Kubecost.                                          |
| Aug 15, 2024     | Kubecost Team updates API documentation and Highlights guidance on deployment guide.                                  |
| Aug 21, 2024     | Kubecost Team requests guidance on feedback on changes in documentation.                                              |
| Aug 22, 2024     | Feedback provided that updating documentation is a strong fix, but not effective as a long term fix to protect users. |
| October 12, 2024 | Report disclosed.                                                                                                     |


### Acknowledgements

I would like to congratulate the Kubecost team for : firstly building a great product that truly solves a problem Kubernetes administrators had for a long time, and secondly for their prompt communications and best efforts to secure the product and the users who deploy it.

