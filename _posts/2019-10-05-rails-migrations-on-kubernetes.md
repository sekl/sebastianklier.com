---
layout: post
title: "Rails Database Migrations on Kubernetes"
date: "2019-10-05T15:15:00.284Z"
categories: [kubernetes, rails]
comments: true
---

If you're running a web app on Kubernetes, no matter if it's Rails, Node or any other tech, you will run into issues that you normally don't have in more traditional environments. One of those is due to the ephemeral nature of your pods and containers and how they are created and destroyed during deployments, which makes things like database migrations a little bit trickier than normal.

<!--more-->

I'm running a Rails app on **Google Kubernetes Engine** which uses replica sets and rolling updates. So when I deploy, a new pod is created first, then Kubernetes will perform readiness probes on this pod and only create more pods and destroy the old ones when my app responds with a 200 status code. My replica set specifies `maxUnavailable: 0`, so if something goes wrong during the rollout the previous version of the code stays available. I've also set `maxSurge: 1`, so only 1 new pod is created at a time.

I use a deployment script to handle building and pushing the Docker image, and `set image` to deploy it to GKE. I then run database migrations on that first available pod, which will cause it to go from a 500 response to the readiness probe (`ActiveRecord::Migration.check_pending!`) to returning 200. Then Kubernetes will create more pods and replicas, take them into the loadbalancer pool and terminate pods running the old code.

This is the script:

``` bash
#!/bin/bash

# Build Docker container and push it to the registry
docker build -t gcr.io/projectname/rails-app:latest .
OUTPUT=$(docker push gcr.io/projectname/rails-app:latest)
echo $OUTPUT

# Parse the output from docker push to get the digest hash
DIGEST=$(echo $OUTPUT | awk '{print $(NF-2)}' | cut -d':' -f 2)
[[ -z "$DIGEST" ]] && { echo "Digest is empty" ; exit 1; }
echo "Digest: ${DIGEST}"

# Label the old pods so we know which ones to target for later commands
kubectl label pods -l app tier=phaseout

# Deploy using the digest hash
kubectl set image deployment/app app=gcr.io/projectname/rails-app@sha256:$DIGEST

# Get the new pods and pick one
PODS=$(kubectl get pods -l 'tier notin (phaseout)' -o name)
echo "New pods for tasks: ${PODS}"
POD=$(echo $PODS | awk '{print $1}' | cut -d'/' -f 2)
[[ -z "$POD" ]] && { echo "Could not find new pod" ; exit 1; }

# The pod will be in status "Creating", so we wait a bit until it's ready
sleep 60

# And finally we can run our command
echo "Connecting to ${POD} to run migrations..."
kubectl exec $POD -c app -- bash -c 'bin/rails db:migrate'

# Confirm we are finished
kubectl rollout status deployment app
```

As long as you pay attention to the order in which you deploy database and code changes, this strategy can be used to achieve minimum disruption to your production service during updates.

There is another way to achieve this, by labeling the new pod instead of the old ones, or targeting it using tags, but I found this way simpler and it does not require changing the version in the deployment manifest.
