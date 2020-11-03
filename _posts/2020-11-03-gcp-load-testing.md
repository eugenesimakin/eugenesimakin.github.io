---
layout: post
title:  "Kubernetes in Google Cloud: Performing load testing using Locust"
tags: gcp kubernetes locust
---

Cloud computing has a lot of benefits for people, and I believe that the future is in Cloud. So, I decided to 
take several online courses to prepare for GCP certification exam. There is a lot to cover, and some
practice wouldn't hurt. Because half of the courses was about GKE (Google Kubernetes Engine), 
I've tried to implement load testing using this guide - 
[Distributed load testing using GKE](https://cloud.google.com/solutions/distributed-load-testing-using-gke).

**Warning!** This is not a guide but just my experience. If you're looking for a guide, please, 
refer to [Distributed load testing using GKE](https://cloud.google.com/solutions/distributed-load-testing-using-gke).

I'll be testing my own PC which will host simple static website. This will be interesting. 
In addition to that, it would be completely legal despite load testing your hosting provider. 

Steps I followed:

1. Create a GCP project.
2. Install Cloud SDK and initialize it.
3. Set environment variables (region, zone, etc.). I choose region in Finland because it's the nearest one.
4. Create the GKE cluster and obtain its credentials.
5. Tweak tasks.py in the sample project at _docker-image/locust-tasks/tasks.py_. On the next step I'll build it.
6. Build the docker image.
7. Deploy the Locust master and worker nodes.
8. Go to the Locust master and perform the test.

Terminal history:
```
export REGION=europe-north1  
export ZONE=${REGION}-b
export PROJECT=$(gcloud config get-value project)
export CLUSTER=gke-load-test
export TARGET="<my_ip>:4000"
export SCOPE="https://www.googleapis.com/auth/cloud-platform"

gcloud config set compute/zone ${ZONE}
gcloud config set project ${PROJECT}
git clone https://github.com/GoogleCloudPlatform/distributed-load-testing-using-kubernetes sample cd sample

gcloud services enable container.googleapis.com

gcloud container clusters create $CLUSTER \
   --zone $ZONE \
   --scopes $SCOPE \
   --enable-autoscaling --min-nodes "3" --max-nodes "10" \
   --scopes=logging-write,storage-ro \
   --addons HorizontalPodAutoscaling,HttpLoadBalancing

gcloud container clusters get-credentials $CLUSTER \
   --zone $ZONE \
   --project $PROJECT

gcloud builds submit \
    --tag gcr.io/$PROJECT/locust-tasks:latest docker-image

# change the yaml files before applying them
kubectl apply -f kubernetes-config/locust-master-controller.yaml
kubectl apply -f kubernetes-config/locust-master-service.yaml
kubectl apply -f kubernetes-config/locust-worker-controller.yaml

# to get status of pods
kubectl get pods -o wide

# to obtain the external IP of the Locust master
kubectl get services

kubectl scale deployment/locust-worker --replicas=20

# clean up after testing (or delete the whole project)
gcloud container clusters delete $CLUSTER --zone $ZONE
```

### Conclusion

My home PC (i7-9700F, 16Gb, 100Mbit/s, nginx) was able to handle several thousand of concurrent users (I tried 1000, 3000 and 6000)
with maximum requests per second value around 1500. More users - more errors and longer response time,
but I didn't observe significant CPU or memory consumption. In my opinion, this setup can
handle 1000 users very well. In this case, average response time will be less than 100ms.
The next step might be to load test more sophisticated web application 
(for example, with authentication and database).

By the way, let's see after a while how much this experiment cost me (cloud computing resources are not free).
