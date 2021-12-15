# openshift-pipelines

## Introduction to OpenShift Pipelines (Tekton)

*OpenShift Pipelines* is a cloud-native, continuous integration and delivery (CI/CD) solution for building pipelines using Tekton. Tekton is a flexible, Kubernetes-native, open-source CI/CD framework that enables automating deployments across multiple platforms (Kubernetes, serverless, VMs, etc) by abstracting away the underlying details.

OpenShift Pipelines features:
•	Standard CI/CD pipeline definition based on Tekton
•	Build images with Kubernetes tools such as S2I, Buildah, Buildpacks, Kaniko, etc
•	Deploy applications to multiple platforms such as Kubernetes, serverless and VMs
•	Easy to extend and integrate with existing tools
•	Scale pipelines on-demand
•	Portable across any Kubernetes platform
•	Designed for microservices and decentralized teams
•	Integrated with the OpenShift Developer Console

This tutorial walks you through pipeline concepts and how to create and run a simple pipeline for building and deploying a containerized app on OpenShift, and in this tutorial, we will use Triggers to handle a real GitHub webhook request to kickoff a PipelineRun.

## In this lab you will:

•	Learn about Tekton concepts
•	Install OpenShift Pipelines
•	Deploy a Sample Application
•	Install Tasks
•	Create a Pipeline
•	Trigger a Pipeline

## Prerequisites
You will need the following to complete the exercises in this lab:
•	Access an OpenShift 4 cluster.
•	You will also use the Tekton CLI (tkn).
 
## Setting up 
Note: The sample output shown in the lab guide may be slightly different than what you see in the output when you issue the commands for your cluster.

### Access the OpenShift web console
1. The OpenShift web console will be opened in a new browser tab. You need to switch between this tab and the new tabs to accomplish many of the lab tasks. You may want to open the new tabs in new windows and display both browser windows at the same time. You may need to disable pop-up blockers if you do not see the new tabs.

2.	In the OpenShift web console, notice the following:
•	The different perspectives available in the OpenShift web console: Administrator and Developer
•	Capabilities of each perspective and dashboards available for each

3.	Take a few minutes to explore the OpenShift web console. Try finding the following:
•	Number of nodes in the cluster inventory dashboard
•	Standard services deployed in the default project
•	Operators installed

4. Connect the cluster with cloudshell or your terminal
- Browse to the OpenShift web console.
- From the dropdown menu in the upper right of the page, click "Copy Login" Command.
- Click the "Display Token" link.
- Copy the "Log in with this token" command line.
- Paste the copied command in your terminal.
- Browse to the oauth token request page and follow the instructions on the page.

Sample command:

    oc login --server https://c109-e.us-east.containers.cloud.ibm.com:31411 -u apikey -p 75ecc28297050904b4fedac2be174595

### Testing your environment
1.	Validate access to your cluster by viewing the nodes in the cluster:
oc get node

Sample Output:
NAME           STATUS   ROLES           AGE   VERSION
10.171.76.77	 Ready    master,worker   37h   v1.16.2

2.	Execute the command below to view services, deployments, and pods:
oc get svc,deploy,po --all-namespaces
Sample Output:
NAME           STATUS   ROLES           AGE   VERSION
10.171.76.77   Ready    master,worker   37h   v1.16.2
container-lab$ oc get svc,deploy,po --all-namespaces
NAMESPACE                                               NAME                                                  TYPE           CLUSTER-IP       EXTERNAL-IP                            PORT(S)                      AGE
calico-system                                           service/calico-typha                                  ClusterIP      172.21.108.58    <none>                                 5473/TCP                     37h
default                                                 service/kubernetes                                    ClusterIP      172.21.0.1       <none>                                 443/TCP                      38h
default                                                 service/openshift                                     ExternalName   <none>           kubernetes.default.svc.cluster.local   <none>                       37h
default                                                 service/openshift-apiserver                           ClusterIP      172.21.146.181   <none>
...

Note: Some commands have large amounts of data in their output. You can scroll up and down in the terminal window and clear it using the clear command.
3.	Execute the command below to clear the terminal screen:
clear
4.	Execute the command below to view all OpenShift projects:
oc get projects
Sample Output:
NAME                                                    DISPLAY NAME             STATUS
calico-system                                                                    Active
default                                                                          Active
example-health                                          Example Health Project   Active
ibm-cert-store                                                                   Active
ibm-system                                                                       Active
kube-node-lease                                                                  Active
kube-public                                                                      Active
kube-system                                                                      Active
openshift                                                                        Active
openshift-apiserver                                                              Active
openshift-apiserver-operator                                                     Active
...

---End Setting up ---
