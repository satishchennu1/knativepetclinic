# createhtpassword
# mavenpipelineopenshift
# petspringdocker
Knative Serving Deployment Model 

Knative Serving controller creates a 
a) Configuration 
b) Revision, 
c) Route

a) microservice as a serverless service on Kubernetes.

Kubernetes resource, a Knative Serving Service can be deployed using a resource YAML file

apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: greeter 1
spec:
  template:
    metadata:
      name: greeter-v1 2
    spec:
      containers:
      - image: quay.io/schennu/ieopetclinic
	  
	  
	  [schennu@beaker-ansible-script-0 knativepetclinic]$ oc get pods
NAME                                      READY   STATUS    RESTARTS   AGE
petclinic-v1-deployment-7976bf488-pnmnw   2/2     Running   0          83s

[schennu@beaker-ansible-script-0 knativepetclinic]$ oc get deployments
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
petclinic-v1-deployment   0/0     0            0           3m46s

[schennu@beaker-ansible-script-0 knativepetclinic]$ oc get ksvc
NAME        URL                                                                 LATESTCREATED   LATESTREADY    READY   REASON
petclinic   http://petclinic-knative-microservice.apps.ocp43.itblab.uspto.gov   petclinic-v1    petclinic-v1   True

oc get services.serving.knative.dev
NAME        URL                                                                 LATESTCREATED   LATESTREADY    READY   REASON
petclinic   http://petclinic-knative-microservice.apps.ocp43.itblab.uspto.gov   petclinic-v1    petclinic-v1   True




[schennu@beaker-ansible-script-0 knativepetclinic]$ oc  get configurations.serving.knative.dev petclinic
NAME        LATESTCREATED   LATESTREADY    READY   REASON
petclinic   petclinic-v1    petclinic-v1   True


[schennu@beaker-ansible-script-0 knativepetclinic]$ oc get routes.serving.knative.dev petclinic
NAME        URL                                                                 READY   REASON
petclinic   http://petclinic-knative-microservice.apps.ocp43.itblab.uspto.gov   True



oc get rev \
 --selector=serving.knative.dev/service=petclinic \
 --sort-by="{.metadata.creationTimestamp}"
 
 [schennu@beaker-ansible-script-0 knativepetclinic]$ oc get configurations petclinic
NAME        LATESTCREATED   LATESTREADY    READY   REASON
petclinic   petclinic-v1    petclinic-v1   True


Step 2:  you want to update the configuration of your existing service  and also ensure you want to rollback the changes if needed

apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: petclinic
spec:
  template:
    metadata:
      name: petclinic-v2
    spec:
      containers:
        - image: quay.io/schennu/ieopetclinic:v2
          env:
          - name: MESSAGE_PREFIX
            value: IEO Game Changer


[schennu@beaker-ansible-script-0 knativepetclinic]$ oc get deployments
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
petclinic-v1-deployment   0/0     0            0           17m
petclinic-v2-deployment   0/1     1            0           8s

watch oc get pods

you will see petclinic-v1 deplpyment had no request for approximately 60 seconds  which is the Knative default scale-down time window. petclinic-v1 was automatically scaled down to zero since it lacked invocations.
 This is the key to how Knative Serving helps you save expensive cloud resources using serverless services
 
 [schennu@beaker-ansible-script-0 knativepetclinic]$ oc get revisions
NAME           CONFIG NAME   K8S SERVICE NAME   GENERATION   READY   REASON
petclinic-v1   petclinic     petclinic-v1       1            True
petclinic-v2   petclinic     petclinic-v2       2            True

[schennu@beaker-ansible-script-0 knativepetclinic]$ oc get rt
NAME        URL                                                                 READY   REASON
petclinic   http://petclinic-knative-microservice.apps.ocp43.itblab.uspto.gov   True
[schennu@beaker-ansible-script-0 knativepetclinic]$
[schennu@beaker-ansible-script-0 knativepetclinic]$
[schennu@beaker-ansible-script-0 knativepetclinic]$ oc get ksvc
NAME        URL                                                                 LATESTCREATED   LATESTREADY    READY   REASON
petclinic   http://petclinic-knative-microservice.apps.ocp43.itblab.uspto.gov   petclinic-v2    petclinic-v2   True
[schennu@beaker-ansible-script-0 knativepetclinic]$ oc get configurations
NAME        LATESTCREATED   LATESTREADY    READY   REASON
petclinic   petclinic-v2    petclinic-v2   True

step 3: 

 may wish to deploy applications using common deployment patterns such as Canary or blue-green. 
 To use these deployment patterns with Knative, you will need to have one or more revisions of the application to distribute the traffic.
 
 distribute the traffic between multiple revesions 
 
 
 oc get ksvc petclinic -o yaml 
 
 
 apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: petclinic
spec:
  template:
    spec:
      containers:
      - image: quay.io/schennu/ieopetclinic:v2
        env:
        - name: MESSAGE_PREFIX
          value: IEO GameChanger
  traffic:
  - tag: current
    revisionName: petclinic-v1
    percent: 100
  - tag: prev
    revisionName: petclinic-v2
    percent: 0
  - tag: latest
    latestRevision: true
    percent: 0



 
  - latestRevision: false
    percent: 100
    revisionName: petclinic-v1
    tag: current
    url: http://current-petclinic-knative-microservice.apps.ocp43.itblab.uspto.gov
  - latestRevision: false
    percent: 0
    revisionName: petclinic-v2
    tag: prev
    url: http://prev-petclinic-knative-microservice.apps.ocp43.itblab.uspto.gov
  - latestRevision: true
    percent: 0
    revisionName: petclinic-s8whh
    tag: latest
    url: http://latest-petclinic-knative-microservice.apps.ocp43.itblab.uspto.gov
  url: http://petclinic-knative-microservice.apps.ocp43.itblab.uspto.gov

 
Step 4 

now we will have canary pattern where you can split the traffic between revisions in increments as small as 1% 



for ((i=1; i<=10; i++)); do
curl http://petclinic-knative-microservice.apps.ocp43.itblab.uspto.gov/ | grep -i "IEO Petclinic Version" 



step 5 : how to do observe knative scale down to zero 



zero auto scaling 



oc project
oc create -f knative-autoscale-service.yaml
oc get pods
hey -c 50 -z 10s \
oc get rt
hey -c 50 -z 10s "http://petclinic-knative-autoscale.apps.ocp43.itblab.uspto.gov/?sleep=3&upto=10000&memload=100"
curl http://petclinic-knative-autoscale.apps.ocp43.itblab.uspto.gov/
hey -c 50 -z 10s "http://petclinic-knative-autoscale.apps.ocp43.itblab.uspto.gov/?sleep=3&upto=100&memload=100"
oc get pods



hey -c 50 -z 10s "http://petclinic-knative-autoscale.apps.ocp43.itblab.uspto.gov/?sleep=3&upto=10000&memload=100"

a) knative Horizontal Pod AutoScaler 





