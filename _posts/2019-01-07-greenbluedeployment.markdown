---
title: "Kubernetes - Blue Green Deployment"
layout: post
date: 2019-01-8 16:41
image: /assets/images/markdown.jpg
headerImage: false
tag:
- kubernetes
- jenkins
category: blog
author: Lefteris Tatakis
description: Kubernetes Blue Green Deployment - Elsevier eCommerse Team
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
---
Its only been a few weeks since using this strategy our production environment at Elsevier, and its already shown its value multiple times! I think its an awesome strategy!
So... what it is?!

Before we start some quick terminology:
 - Blue deployment is the deployment currently in production
 - Green deployment is the new release we want to make


## Overview - 10000 ft ✈️
*Blue-green deployment* is a strategy that allows you to switch deployments of your code with minimal, dare I say 0 down time.
The code is versioned, we call the old code the blue deployment while the new one green. The versioning of the code ensures that the traffic is routed to the *old* pods initially, until, we are confident that the new code is ready to be deployed! 
Once we are happy that the `Green deployment` is up and running, we switch the traffic and we delete the blue deployment.

![Markdowm Image][1]
<figcaption class="caption">Starting point, traffic going to blue deployment</figcaption>

<p align="center">Voila! </p>

## The problem we solved
We noticed that in our Production system the initial 5-10 minutes of the new deployment was unstable, eg we saw old code showing between refreshes of the pages.
This was because, until the new code was up and running we had both deployments of our code serving our customers, obviously causing instability and weird behavior.

> Each release deployment is versioned.


![Markdowm Image][2]
<figcaption class="caption">Following successful green deployment traffic</figcaption>


## The meat 🥩 of it!
The deployment YML looked like:
```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: the-awesome-page-##VERSION##
spec:
  selector:
    matchLabels:
      app: the-awesome-page
      version: ##VERSION##
  replicas: 3
  template:
    metadata:
      labels:
        app: the-awesome-page
        version: ##VERSION##
    spec:
      containers:
      - name: the-awesome-page
        image: ##IMAGE##
    ...
```

The service of the application
```YAML
apiVersion: v1
kind: Service
metadata:
  name: the-awesome-page
spec:
  ports:
  - port: 80
    name: http
    targetPort: http
  selector:
    app: the-awesome-page
    version: ##VERSION##
```

and the implementation strategy was done in the Jenkins pipeline step:
```sh
BLUEDEPLOYMENT=$(kubectl --namespace ${namespace} get service ${project} -o json | jq '.spec.selector.version')

cd kubernetes
for f in *.yml
do
    if [ "$f" != "service.yml" ] ; then
        kubectl --namespace=${namespace} apply -f "$f"
    fi
done
cd ..

READY=$(kubectl get deploy ${project}-${buildTag} -o json --namespace=${namespace} | jq '.status.conditions[] | select(.reason == "MinimumReplicasAvailable") | .status' | tr -d '"')

TIMEOUT=0
while [[ "$READY" != "True" && "$TIMEOUT" -lt 30 ]]; do
    READY=$(kubectl get deploy ${project}-${buildTag} -o json --namespace=${namespace} | jq '.status.conditions[] | select(.reason == "MinimumReplicasAvailable") | .status' | tr -d '"')
    echo "The green blue deployment is not ready yet"
    sleep 10
    TIMEOUT=$((TIMEOUT + 1))
done

kubectl --namespace=${namespace} apply -f ./kubernetes/service.yml
GREENDEPLOYMENT=$(kubectl --namespace ${namespace} get service ${project} -o json | jq '.spec.selector.version')

if [ ! -z "$BLUEDEPLOYMENT" ] && [ "$BLUEDEPLOYMENT" != "$GREENDEPLOYMENT" ] ; then
    { # try - we did this in case the old deployment was not there anymore for some reason
        BLUEDEPLOYMENT=$(echo $BLUEDEPLOYMENT | tr -d '"')
        kubectl delete deployment ${project}-$BLUEDEPLOYMENT --namespace=${namespace}
    } || { # catch
        # save log for exception
    }
fi
```
## Details of the implementation 
### Retrieving Blue version
First, we need to retrieve the version that is currently used in the `Blue deployment`.
The module that controls this routing is the `Service` of our deployment in Kubernetes.
We do this by asking Kubernetes to give us the current version of the deployment its using.
```sh
BLUEDEPLOYMENT=$(kubectl --namespace ${namespace} get service ${project} -o json | jq '.spec.selector.version')
```

### YML Deployment Sequence
Previously, our approach was to apply all the YML in one go, for example: 
```sh
kubectl apply -f ./
```
However, to get the benefit of this strategy we apply our `service.yml` only once the rest of the deployment is up.
This is because if we apply _all_ the YML at once the `Service` will be overridden straight away, causing bigger problems than we are trying to solve!

Therefore, we initially  deploy the deployment and the ingresses:
```sh
kubectl --namespace=${namespace} apply -f ./kubernetes/deployment.yml
kubectl --namespace=${namespace} apply -f ./kubernetes/ingress.yml
```
and only once all the required deployments are in place, we deploy the service new service:
```sh
kubectl --namespace=${namespace} apply -f ./kubernetes/service.yml
```

### Ensuring the code is successfully  deployed
In order to check the code is there and ready to be served to our customers, we loop through the condition.
The condition that is responsible for us knowing if we are ready is:
```sh
READY=$(kubectl get deploy ${project}-${version} -o json --namespace=${namespace} | jq '.status.conditions[] | select(.reason == "MinimumReplicasAvailable") | .status' | tr -d '"')
```
What are retrieving the Kubernetes information for the deployment we just made and make sure we have the `MinimumReplicasAvailable` needed for this deployment.
If this criterion is not met, we are not ready to route traffic through, so we wait and try again later ✌️.


### Deleting of the old deployment
When implementing this we needed to be aware of two key possible issues:

1) The blue and green deployments are the same. This would occur if we are rerunning the Jenkins job that deployed the current version.  


2) The old deployment does not exist anymore. This is not a scenario we should see in production, but in `dev` where we frequently go and delete a deployment due to _X_ or _Y_ reason. We handle this by not failing the Jenkins build because we don't want the Jenkins build to fail, as the latest deployment has been successful at this point!

Therefore, we added the comparison of the `blue` and `green` version to combat point (1), and we created a try/catch block to ensure we don't fail our build due to some behind the scenes manual work, for (2).

## Conclusion
Blue-green deployment is a strategy that allows minimal down time of your application. Its straightforward to implement and very valuable in my opinion.

Thank you for reading! 👋

If you notice any mistakes or spelling mistakes please contact me on <a href="https://twitter.com/LTatakis"> Twitter</a>.

[1]: /assets/kubernetes/point2blue.png
[2]: /assets/kubernetes/point2green.png
