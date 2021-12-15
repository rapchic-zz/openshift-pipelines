# openshift-pipelines

## Introduction to OpenShift Pipelines (Tekton)

*OpenShift Pipelines* is a cloud-native, continuous integration and delivery (CI/CD) solution for building pipelines using Tekton. Tekton is a flexible, Kubernetes-native, open-source CI/CD framework that enables automating deployments across multiple platforms (Kubernetes, serverless, VMs, etc) by abstracting away the underlying details.

OpenShift Pipelines features:
-	Standard CI/CD pipeline definition based on Tekton
-	Build images with Kubernetes tools such as S2I, Buildah, Buildpacks, Kaniko, etc
-	Deploy applications to multiple platforms such as Kubernetes, serverless and VMs
-	Easy to extend and integrate with existing tools
-	Scale pipelines on-demand
-	Portable across any Kubernetes platform
-	Designed for microservices and decentralized teams
-	Integrated with the OpenShift Developer Console

This tutorial walks you through pipeline concepts and how to create and run a simple pipeline for building and deploying a containerized app on OpenShift, and in this tutorial, we will use Triggers to handle a real GitHub webhook request to kickoff a PipelineRun.

## In this lab you will:

-	Learn about Tekton concepts
-	Install OpenShift Pipelines
-	Deploy a Sample Application
-	Install Tasks
-	Create a Pipeline
-	Trigger a Pipeline

## Prerequisites
You will need the following to complete the exercises in this lab:
-	Access an OpenShift 4 cluster.
-	You will also use the Tekton CLI (tkn).
 
## Setting up 
Note: The sample output shown in the lab guide may be slightly different than what you see in the output when you issue the commands for your cluster.

### Access the OpenShift web console
1. The OpenShift web console will be opened in a new browser tab. You need to switch between this tab and the new tabs to accomplish many of the lab tasks. You may want to open the new tabs in new windows and display both browser windows at the same time. You may need to disable pop-up blockers if you do not see the new tabs.

2.	In the OpenShift web console, notice the following:
-	The different perspectives available in the OpenShift web console: Administrator and Developer
-	Capabilities of each perspective and dashboards available for each

3.	Take a few minutes to explore the OpenShift web console. Try finding the following:
-	Number of nodes in the cluster inventory dashboard
-	Standard services deployed in the default project
-	Operators installed

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

``` oc get node ```

Sample Output:
```
NAME           STATUS   ROLES           AGE   VERSION
10.171.76.77	 Ready    master,worker   37h   v1.16.2 
```

2.	Execute the command below to view services, deployments, and pods:

``` oc get svc,deploy,po --all-namespaces ```

Sample Output:

```
NAME           STATUS   ROLES           AGE   VERSION
10.171.76.77   Ready    master,worker   37h   v1.16.2
container-lab$ oc get svc,deploy,po --all-namespaces
NAMESPACE                                               NAME                                                  TYPE           CLUSTER-IP       EXTERNAL-IP                            PORT(S)                      AGE
calico-system                                           service/calico-typha                                  ClusterIP      172.21.108.58    <none>                                 5473/TCP                     37h
default                                                 service/kubernetes                                    ClusterIP      172.21.0.1       <none>                                 443/TCP                      38h
default                                                 service/openshift                                     ExternalName   <none>           kubernetes.default.svc.cluster.local   <none>                       37h
default                                                 service/openshift-apiserver                           ClusterIP      172.21.146.181   <none>
...
```

Note: Some commands have large amounts of data in their output. You can scroll up and down in the terminal window and clear it using the clear command.
 
3.	Execute the command below to clear the terminal screen:
 
``` clear ```
 
4.	Execute the command below to view all OpenShift projects:
``` oc get projects ```

Sample Output:
```
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
```
---End Setting up ---


## Exercise 1
Learn about Tekton concepts

Tekton defines a number of Kubernetes custom resources as building blocks in order to standardize pipeline concepts and provide a terminology that is consistent across CI/CD solutions. These custom resources are an extension of the Kubernetes API that let users create and interact with these objects using kubectl and other Kubernetes tools.

The custom resources needed to define a pipeline are listed below:
-	Task: a reusable, loosely coupled number of steps that perform a specific task (e.g. building a container image)
-	Pipeline: the definition of the pipeline and the Tasks that it should perform
-	TaskRun: the execution and result of running an instance of task
-	PipelineRun: the execution and result of running an instance of pipeline, which includes a number of TaskRuns

<img width="800" alt="image" src="https://user-images.githubusercontent.com/6327371/146123336-cd4b7f6a-7ad2-4f00-b61c-559a16d2d2c2.png">

For further details on pipeline concepts, refer to the Tekton documentation that provides an excellent guide for understanding various parameters and attributes available for defining pipelines.

The Tekton API enables functionality to be separated from configuration (e.g. Pipelines vs PipelineRuns) such that steps can be reusable.
Triggers extends the Tekton architecture with the following CRDs:
-	**TriggerTemplate** - Templates resources to be created (e.g. Create PipelineResources and PipelineRun that uses them)
-	**TriggerBinding** - Validates events and extracts payload fields
-	**EventListener** - Connects TriggerBindings and TriggerTemplates into an addressable endpoint (the event sink). It uses the extracted event parameters from each TriggerBinding (and any supplied static parameters) to create the resources specified in the corresponding TriggerTemplate. It also optionally allows an external service to pre-process the event payload via the interceptor field.
-	**ClusterTriggerBinding** - A cluster-scoped TriggerBinding
Using tektoncd/triggers in conjunction with tektoncd/pipeline enables you to easily create full-fledged CI/CD systems where the execution is defined entirely through Kubernetes resources.

Using *tektoncd/triggers* in conjunction with tektoncd/pipeline enables you to easily create full-fledged CI/CD systems where the execution is defined entirely through Kubernetes resources.
You can learn more about triggers by checking out the [docs](https://github.com/tektoncd/triggers/blob/master/docs/README.md).

In the following sections, you will go through each of the above steps to define and invoke a pipeline.

--End of Exercise 1--

## Exercise 2

### Install OpenShift Pipelines

Install OpenShift Pipelines
OpenShift Pipelines is provided as an add-on on top of OpenShift that can be installed via an operator available in the OpenShift OperatorHub.

<img width="750" alt="image" src="https://user-images.githubusercontent.com/6327371/146124651-6698c972-9c5c-4aa1-96f3-55ed34d2848d.png">

To start, make sure you are on the Administrator perspective as shown below:

<img width="300" alt="image" src="https://user-images.githubusercontent.com/6327371/146124670-5ab79855-cad4-45a0-a6b7-8e6dcafb1986.png">

Go to *Operators > OperatorHub* in the Web Console. You can see the list of available operators for OpenShift provided by Red Hat as well as a community of partners and open-source projects.

<img width="600" alt="image" src="https://user-images.githubusercontent.com/6327371/146124715-95b6cdf4-a378-49a8-a16e-cee1911bfb69.png">

In the search bar where it says Filter by keyword..., type *OpenShift Pipelines* to find the OpenShift Pipelines Operator:

<img width="650" alt="image" src="https://user-images.githubusercontent.com/6327371/146124772-1962f5cb-eaf4-42d2-95c2-7cb2ed8ed908.png">

Click on *OpenShift Pipelines Operator*, Continue, and then Install:

<img width="550" alt="image" src="https://user-images.githubusercontent.com/6327371/146124816-ee4a9fc4-b8e4-417e-b61d-38970e7568bb.png">

Leave the default settings and click on *Install* in order to install the Operator:

<img width="600" alt="image" src="https://user-images.githubusercontent.com/6327371/146124851-272e1f46-5b2b-4af0-b55f-de2503720ae0.png">

After clicking Install, you will be taken to the Installed Operators page. If you do not see the OpenShift Pipelines Operator as shown below, simply wait a moment while the OpenShift Pipelines Operator finishes installation:

<img width="780" alt="image" src="https://user-images.githubusercontent.com/6327371/146124904-e0b0801a-d53d-46da-98a7-d44161d8dac1.png">

That's all. The operator now installs OpenShift Pipelines on the cluster.
You can confirm the following by checking tekton-pipelines and tekton-triggers pods with Running state in openshift-pipelines namespace. If so, openshift-pielines have been installed on your cluster.

<img width="600" alt="image" src="https://user-images.githubusercontent.com/6327371/146124926-f77e1f2a-6090-4efe-bfce-e78e7214dbcb.png">

-- End of Exercise 2 --

## Exercise 3

### Create a project

From the terminal/shell 

Create a project for the sample application that you will be using in this tutorial:

    oc new-project pipelines-tutorial

OpenShift Pipelines automatically adds and configures a ServiceAccount named pipeline that has sufficient permissions to build and push an image. This service account will be used later in the tutorial.

Run the following command to see the pipeline service account:

    oc get serviceaccount pipeline

You will use the simple application during this tutorial, which has a frontend and backend
You can also deploy the same applications by applying the artifacts available in k8s directory of the respective repo.

If you deploy the application directly, you should be able to see the deployment in the OpenShift Web Console by switching over to the Developer perspective of the OpenShift Web Console. Change from Administrator to Developer from the drop down as shown below:
 
Make sure you are on the pipelines-tutorial project by selecting it from the Project dropdown menu. Either search for pipelines-tutorial in the search bar or scroll down until you find pipelines-tutorial and click on the name of your project.


<img width="800" alt="image" src="https://user-images.githubusercontent.com/6327371/146125166-c1a53212-03a2-4e8d-a3be-2ef3a0fbd905.png">

-- End of Exercise 3 --

## Exercise 4

### Create Pipeline Tasks

Here is an example of a Maven task for building a Maven-based Java application:

    apiVersion: tekton.dev/v1beta1
    kind: Task
    metadata:
      name: maven-build
    spec:
      resources:
        inputs:
        - name: workspace-git
          targetPath: /
          type: git
      steps:
      - name: build
        image: maven:3.6.0-jdk-8-slim
        command:
        - /usr/bin/mvn
        args:
        - install
You can find more examples of reusable tasks in the Tekton Catalog and [OpenShift Catalog](https://github.com/openshift/pipelines-catalog) repositories.

Install the *apply-manifests* and *update-deployment tasks* from the repository using oc or kubectl, which you will need for creating a pipeline in the next section:

    oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/01_pipeline/01_apply_manifest_task.yaml

    oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/01_pipeline/02_update_deployment_task.yaml

You can take a look at the tasks you created using the Tekton CLI:

    tkn task ls

resulting in output similar to:

    $ tkn task ls
    NAME                AGE
    apply-manifests     10 seconds ago
    update-deployment   4 seconds ago

We will be using *buildah clusterTasks*, which gets installed along with the Pipeline Operator. The Operator installs a few ClusterTask which you can see.

    tkn clustertasks ls

resulting in output similar to:

    $ tkn clustertasks ls
    NAME                       DESCRIPTION   AGE
    buildah                                  1 day ago
    buildah-v0-14-3                          1 day ago
    git-clone                                1 day ago
    s2i-php                                  1 day ago
    tkn                                      1 day ago

---End of Exercise 4 ---

## Exercise 5

### Prepare the Pipeline

In the next step, we will define a Pipeline, which will be used to build an application image from an arbitrary GitHub repository and deploy the image as a running instance.

It'll look like this:

    apiVersion: tekton.dev/v1beta1
    kind: Pipeline
    metadata:
      name: build-and-deploy
    spec:
      workspaces:
      - name: shared-workspace
      params:
      - name: deployment-name
        type: string
        description: name of the deployment to be patched
      - name: git-url
        type: string
        description: url of the git repo for the code of deployment
      - name: git-revision
        type: string
        description: revision to be used from repo of the code for deployment
        default: "master"
      - name: IMAGE
        type: string
        description: image to be build from the code
      tasks:
      - name: fetch-repository
        taskRef:
          name: git-clone
          kind: ClusterTask
        workspaces:
        - name: output
          workspace: shared-workspace
        params:
        - name: url
          value: $(params.git-url)
        - name: subdirectory
          value: ""
        - name: deleteExisting
          value: "true"
        - name: revision
          value: $(params.git-revision)
      - name: build-image
        taskRef:
          name: buildah
          kind: ClusterTask
        params:
        - name: TLSVERIFY
          value: "false"
        - name: IMAGE
          value: $(params.IMAGE)
        workspaces:
        - name: source
          workspace: shared-workspace
        runAfter:
        - fetch-repository
      - name: apply-manifests
        taskRef:
          name: apply-manifests
        workspaces:
        - name: source
          workspace: shared-workspace
        runAfter:
        - build-image
      - name: update-deployment
        taskRef:
          name: update-deployment
        workspaces:
        - name: source
          workspace: shared-workspace
        params:
        - name: deployment
          value: $(params.deployment-name)
        - name: IMAGE
          value: $(params.IMAGE)
        runAfter:
        - apply-manifests


You need to create the PersistentVolumeClaim which can be used for Pipeline execution:

    oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/01_pipeline/03_persistent_volume_claim.yaml

Run the following command to check when the volume is available:

    oc get pvc

The resulting status should change from:

    $ oc get pvc
    NAME         STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS      AGE
    source-pvc   Pending 

to

        $ oc get pvc
        NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
        source-pvc   Bound    
    
    pvc-5a9e5699-fdfb-468c-b0bf-c85b37c39bd0   20Gi       RWO            ibmc-block-gold   83s

--- End of Exercise 5 ---

