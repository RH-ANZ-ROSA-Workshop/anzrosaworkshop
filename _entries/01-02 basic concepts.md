---
sectionid: concepts
sectionclass: h2
title: Basic concepts
parent-id: intro
---

### ROSA CLI Demonstration & ROSA example setup

In this section, we’ll give you some links that can be used after the class for independent study.  We’ll demonstrate how to use some ROSA commands, but due to the nature of our shared environment for this workshop, not every student will be able to log in with the full permissions required.  When you test this on your own with your own AWS credentials, all the commands will work in the same manner as they have been demonstrated today.

To effectively administer a ROSA cluster, you’ll use a combination of “rosa,” “oc,” and “aws” command line commands.  Below is a concise list of some of the more commonly used ROSA command-line commands which we will be demonstrating today.  Later in the hands-on section of the workshop, you will be using some of the “oc” commands to deploy an application and query its status. The complete ROSA command list can be found at the following location: [https://docs.openshift.com/rosa/rosa_cli/rosa-get-started-cli.html](https://docs.openshift.com/rosa/rosa_cli/rosa-get-started-cli.html)

> **NOTE:** You will need to get your individual ROSA token to use when you practice this independently.  Instructions can be found in the section “Configuring the ROSA CLI” in the Get Started link above.

#### ROSA CLI Demonstration

Here are some of the more commonly used ROSA commands.

{% collapsible %}

rosa login [arguments] (to login)

rosa verify permissions [arguments] (e.g. rosa verify permissions --region=ap-southeast-2) - (to verify the AWS permissions are set correctly for the specified region)

rosa verify quota [arguments] (to verify the AWS quotas are set correctly for the region)

rosa download oc (to download the OpenShift oc client software)

rosa verify oc - (to verify the OpenShift oc client software has been installed and is available)

rosa whoami [arguments] (to get the currently logged in user and environment details)

rosa version [arguments] (to get the version of the installed ROSA software)

rosa create cluster --cluster=<cluster_name> --debug (to create a cluster with the specified name.  NB: there are MANY options to this command and a thorough review of the documentation is recommended.)

rosa list users --cluster=<cluster_name> (to get a list of users associated with the specified cluster)

rosa list clusters (to list current clusters)

rosa describe cluster --cluster=<cluster_name> (to get details of specified cluster)

{% endcollapsible %}

####  ROSA example setup

Further below, we’ve also provided links to both the official ROSA video on the Red Hat website and a short unofficial YouTube video created by an Australian-based Red Hat employee that demonstrates an abbreviated version of how to set up ROSA yourself with your own AWS credentials for your independent study.

Official ROSA video - [https://www.youtube.com/watch?v=MFcbuxkP3C4](https://www.youtube.com/watch?v=MFcbuxkP3C4)

ROSA setup - [https://youtu.be/l7ylYBP8p4Q](https://youtu.be/l7ylYBP8p4Q)

For reference, the below section is an extraction of commands used in the ROSA setup video above.

{% collapsible %}

AWS UI - create EC2 machine (t2.medium) & create/re-use key-pair

AWS UI - rename cluster

Terminal - ssh ec2-user@<IPAddress> -i /Users/<username>/Downloads/<key-pair>.pem

sudo su

hostnamectl set-hostname <hostname>

Exit, exit & relogin & sudo su

aws configure & put in access key & secret access key & region & output format

console.redhat.com/openshift/downloads - DL the ROSA CLI (choosing the right OS type first)

aws sts get-caller-identity

aws ec2 describe-instances

tar -xvf ROSACLI.tar & mv rosa /usr/bin

rosa –help (or just “rosa” or “rosa -h”) to show list of arguments

https://console.redhat.com/openshift/token/rosa - download token

rosa login & paste token contents into prompt.

rosa verify permissions

rosa verify quota

rosa init

rosa download oc

tar -xvf OCCLI.tar & mv oc /usr/bin & mv kubectl /usr/bin

rosa verify oc

rosa create cluster --cluster-name=<ROSA-example>

rosa list clusters

Rosa logs install -c <ROSA-example> --watch (to watch cluster install logs)

URL at end: https://console.redhat.com/openshift/details/s/<guid>

Terminal: rosa create admin --cluster=<cluster_name>

Get login cmd from results (e.g. - oc login https://api.<cluster-name>.vpbx.p1.openshiftapps.com:6443 --username cluster-admin --password abcde-fghij-klmno-pqrst

In GUI, under Clusters click “Open Console” on right and login w/ cluster-admin/<password from last step>

Terminal: watch oc get pods -n openshift-logging

AWS Console: go to CloudWatch to watch status

Terminal: rosa list addons --cluster=<cluster-name>

{% endcollapsible %}

Link to alternative ROSA workshop with more advanced setup steps - [Red Hat OpenShift on AWS Workshop](http://rosaworkshop.io)

### Source-To-Image (S2I)

Source-to-Image (S2I) is a toolkit and workflow for building reproducible container images from source code. S2I produces ready-to-run images by injecting source code into a container image and letting the container prepare that source code for execution. By creating self-assembling builder images, you can version and control your build environments exactly like you use container images to version your runtime environments.

#### How it works

{% collapsible %}

For a dynamic language like Ruby, the build-time and run-time environments are typically the same. Starting with a builder image that describes this environment - with Ruby, Bundler, Rake, Apache, GCC, and other packages needed to set up and run a Ruby application installed - source-to-image performs the following steps:

1. Start a container from the builder image with the application source injected into a known directory

1. The container process transforms that source code into the appropriate runnable setup - in this case, by installing dependencies with Bundler and moving the source code into a directory where Apache has been preconfigured to look for the Ruby config.ru file.

1. Commit the new container and set the image entrypoint to be a script (provided by the builder image) that will start Apache to host the Ruby application.

For compiled languages like C, C++, Go, or Java, the dependencies necessary for compilation might dramatically outweigh the size of the actual runtime artifacts. To keep runtime images slim, S2I enables a multiple-step build processes, where a binary artifact such as an executable or Java WAR file is created in the first builder image, extracted, and injected into a second runtime image that simply places the executable in the correct location for execution.

For example, to create a reproducible build pipeline for Tomcat (the popular Java webserver) and Maven:

1. Create a builder image containing OpenJDK and Tomcat that expects to have a WAR file injected

1. Create a second image that layers on top of the first image Maven and any other standard dependencies, and expects to have a Maven project injected

1. Invoke source-to-image using the Java application source and the Maven image to create the desired application WAR

1. Invoke source-to-image a second time using the WAR file from the previous step and the initial Tomcat image to create the runtime image

By placing our build logic inside of images, and by combining the images into multiple steps, we can keep our runtime environment close to our build environment (same JDK, same Tomcat JARs) without requiring build tools to be deployed to production.

{% endcollapsible %}

#### Goals and benefits

{% collapsible %}

##### Reproducibility

Allow build environments to be tightly versioned by encapsulating them within a container image and defining a simple interface (injected source code) for callers. Reproducible builds are a key requirement to enabling security updates and continuous integration in containerized infrastructure, and builder images help ensure repeatability as well as the ability to swap runtimes.

##### Flexibility

Any existing build system that can run on Linux can be run inside of a container, and each individual builder can also be part of a larger pipeline. In addition, the scripts that process the application source code can be injected into the builder image, allowing authors to adapt existing images to enable source handling.

##### Speed

Instead of building multiple layers in a single Dockerfile, S2I encourages authors to represent an application in a single image layer. This saves time during creation and deployment, and allows for better control over the output of the final image.

##### Security

Dockerfiles are run without many of the normal operational controls of containers, usually running as root and having access to the container network. S2I can be used to control what permissions and privileges are available to the builder image since the build is launched in a single container. In concert with platforms like OpenShift, source-to-image can enable admins to tightly control what privileges developers have at build time.

{% endcollapsible %}

### Routes

An OpenShift `Route` exposes a service at a host name, like www.example.com, so that external clients can reach it by name. When a `Route` object is created on OpenShift, it gets picked up by the built-in HAProxy load balancer in order to expose the requested service and make it externally available with the given configuration. You might be familiar with the Kubernetes `Ingress` object and might already be asking "what's the difference?". Red Hat created the concept of `Route` in order to fill this need and then contributed the design principles behind this to the community; which heavily influenced the `Ingress` design.  Though a `Route` does have some additional features as can be seen in the chart below.

![routes vs ingress](media/managedlab/routes-vs-ingress.png)

> **NOTE:** DNS resolution for a host name is handled separately from routing; your administrator may have configured a cloud domain that will always correctly resolve to the router, or if using an unrelated host name you may need to modify its DNS records independently to resolve to the router.

Also of note is that an individual route can override some defaults by providing specific configuraitons in its annotations.  See here for more details: [https://docs.openshift.com/dedicated/architecture/networking/routes.html#route-specific-annotations](https://docs.openshift.com/dedicated/architecture/networking/routes.html#route-specific-annotations)

### ImageStreams

An ImageStream stores a mapping of tags to images, metadata overrides that are applied when images are tagged in a stream, and an optional reference to a Docker image repository on a registry.


#### What are the benefits? 

{% collapsible %}

Using an ImageStream makes it easy to change a tag for a container image.  Otherwise to change a tag you need to download the whole image, change it locally, then push it all back. Also promoting applications by having to do that to change the tag and then update the deployment object entails many steps.  With ImageStreams you upload a container image once and then you manage it’s virtual tags internally in OpenShift.  In one project you may use the `dev` tag and only change reference to it internally, in prod you may use a `prod` tag and also manage it internally. You don't really have to deal with the registry!

You can also use ImageStreams in conjuction with DeploymentConfigs to set a trigger that will start a deployment as soon as a new image appears or a tag changes its reference.


{% endcollapsible %}


See here for more details: [https://blog.openshift.com/image-streams-faq/](https://blog.openshift.com/image-streams-faq/) <br>
OpenShift Docs: [https://docs.openshift.com/container-platform/3.11/dev_guide/managing_images.html](https://docs.openshift.com/container-platform/3.11/dev_guide/managing_images.html)<br>
ImageStream and Builds: [https://cloudowski.com/articles/why-managing-container-images-on-openshift-is-better-than-on-kubernetes/](https://cloudowski.com/articles/why-managing-container-images-on-openshift-is-better-than-on-kubernetes/)


### Builds

A build is the process of transforming input parameters into a resulting object. Most often, the process is used to transform input parameters or source code into a runnable image. A BuildConfig object is the definition of the entire build process.

OpenShift Container Platform leverages Kubernetes by creating Docker-formatted containers from build images and pushing them to a container image registry.

Build objects share common characteristics: inputs for a build, the need to complete a build process, logging the build process, publishing resources from successful builds, and publishing the final status of the build. Builds take advantage of resource restrictions, specifying limitations on resources such as CPU usage, memory usage, and build or pod execution time.

See here for more details: [https://docs.openshift.com/container-platform/3.11/architecture/core_concepts/builds_and_image_streams.html](https://docs.openshift.com/container-platform/3.11/architecture/core_concepts/builds_and_image_streams.html)
