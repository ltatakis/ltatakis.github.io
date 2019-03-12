---
title: "Kubernetes Node Recycle"
layout: post
date: 2019-03-12 13:40
image: /assets/kubernetes/nick.png
headerImage: false
tag:
- kubernetes
- commands
category: blog
description: "Kubernetes Node Recycle commands"
author: LefterisTatakis
---
Recycling nodes in Kubernetes is quite easy (given there is enough capacity to deal with the Pods of the old Node) 🤓

Commands the need to do:
```bash
    kubectl cordon ${NODE_NAME}
```
```bash
    kubectl drain --ignore-daemonsets --delete-local-data ${NODE_NAME}
```
```bash
    kubectl delete node ${NODE_NAME}
```
Once the Kubernetes delete is done, if you use AWS EC2 instances to host the Nodes as we do, you will have to go to the ASG you have defined and then un-attach and destroy that EC2 instance.

For example, you could do the following command in a `for` loop to upgrade your Kubernetes cluster with a new AMI image for your Node.

Thank you for reading! 👋

If you notice any mistakes please contact me on <a href="https://twitter.com/LTatakis"> Twitter</a>.

[1]: /assets/images/ml/MLinAnImage.png
[2]: /assets/images/ml/petclassification.png
[4]: /assets/images/ml/neural.png
[3]: /assets/images/ml/overviewOfMLProcess.png
