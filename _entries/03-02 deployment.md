---
sectionid: lab2-app-deployment
sectionclass: h2
title: Application Deployment
parent-id: lab-clusterapp
---

### Create new project

Create a new project called "OSToy" in your cluster.

{% collapsible %}

Use the following command

`oc new-project ostoy<student#>`

You should receive the following response

```sh
$ oc new-project ostoy<student#>
Now using project "ostoy<student#>" on server "https://api.gz49n8jb.westeurope.rosaapp.io:6443".
```

> **Hint** You can add applications to this project with the 'new-app' command. For example, try:
>
> `oc new-app rails-postgresql-example`
>
> To build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:
>
> `kubectl create deployment hello-node --image=k8s.gcr.io/serve_hostname`
>

Equivalently, you can also create this new project using the web UI by selecting **Home -> Projects**, then clicking on the **Create Project** button on the right.

![UI Create Project](media/managedlab/6-ostoy-newproj.png)

{% endcollapsible %}

### Download YAML configuration

Download the Kubernetes deployment object yamls from the following locations to jump host to a directory of your choosing - just remember where you placed them for the next step.

`curl -O https://raw.githubusercontent.com/RH-ANZ-Workshops/anzworkshop/main/yaml/ostoy-fe-deployment.yaml`

`curl -O https://raw.githubusercontent.com/RH-ANZ-Workshops/anzworkshop/main/yaml/ostoy-microservice-deployment.yaml`

{% collapsible %}

Feel free to open them up and take a look at what we will be deploying. For simplicity of this lab we have placed all the Kubernetes objects we are deploying in one "all-in-one" yaml file.  Though in reality there are benefits to separating these out into individual yaml files.

[ostoy-fe-deployment.yaml](https://github.com/RH-ANZ-Workshops/anzworkshop/blob/main/yaml/ostoy-fe-deployment.yaml)

[ostoy-microservice-deployment.yaml](https://github.com/RH-ANZ-Workshops/anzworkshop/blob/main/yaml/ostoy-microservice-deployment.yaml)

{% endcollapsible %}

### Deploy backend microservice

The microservice application serves internal web requests and returns a JSON object containing the current hostname and a randomly generated color string.

{% collapsible %}

In your command line deploy the microservice using the following command:

`oc apply -f ostoy-microservice-deployment.yaml`

You should see the following response:
```
$ oc apply -f ostoy-microservice-deployment.yaml
deployment.apps/ostoy-microservice created
service/ostoy-microservice-svc created
```

{% endcollapsible %}

### Deploy the front-end service

The frontend deployment contains the node.js frontend for our application along with a few other Kubernetes objects to illustrate examples.

{% collapsible %}

 If you open the *ostoy-fe-deployment.yaml* you will see we are defining:

- Persistent Volume Claim
- Deployment Object
- Service
- Route
- Configmaps
- Secrets

In your command line deploy the frontend along with creating all objects mentioned above by entering:

`oc apply -f ostoy-fe-deployment.yaml`

You should see all objects created successfully

```sh
$ oc apply -f ostoy-fe-deployment.yaml
persistentvolumeclaim/ostoy-pvc created
deployment.apps/ostoy-frontend created
service/ostoy-frontend-svc created
route.route.openshift.io/ostoy-route created
configmap/ostoy-configmap-env created
secret/ostoy-secret-env created
configmap/ostoy-configmap-files created
secret/ostoy-secret created
```

{% endcollapsible %}

### Get route

Get the route so that we can access the application via `oc get route`

{% collapsible %}

You should see the following response:

```sh
NAME          HOST/PORT                                                      PATH      SERVICES              PORT      TERMINATION   WILDCARD
ostoy-route   ostoy-route-ostoy01.apps.qv4g35sq.westeurope.rosaapp.io                   ostoy-frontend-svc    <all>                   None
```

Copy `ostoy-route-ostoy<student#>.apps.qv4g35sq.westeurope.rosaapp.io` from the command line and paste it into your browser and press enter.  You should see the homepage of our application.

![Home Page](media/managedlab/10-ostoy-homepage.png)

{% endcollapsible %}
