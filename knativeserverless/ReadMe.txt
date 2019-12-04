Installing the OpenShift Serverless Operator
Note: The OpenShift Serverless Operator only works for OpenShift Container Platform versions 4.1.13 and later.

Installing Knative Serving

a) You must create a KnativeServing object to install Knative Serving using the OpenShift Serverless Operator.

b) sampleserving.yaml 

apiVersion: v1
kind: Namespace
metadata:
 name: knative-serving
---
apiVersion: serving.knative.dev/v1alpha1
kind: KnativeServing
metadata:
 name: knative-serving
 namespace: knative-serving


prerequisite 
a) an account with Cluster Administrator Access .

b) Installed Openshift Serverless Operator 


c) oc apply -f serving.yaml 

d) Verify the installation is complete by using the command

   $ oc get knativeserving/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'

Results should be similar to: 
   DeploymentsAvailable=True
   InstallSucceeded=True
   Ready=True

