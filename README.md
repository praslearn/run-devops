# Whats Including In This Repository

This is a complete example of implementation of a microservices-based architecture available for studying. This source code was forked and adapted from the repository [course](https://www.udemy.com/course/microservices-architecture-and-implementation-on-dotnet/?couponCode=AUGUST2021).

For more details about the application, please see this [link](https://github.com/aspnetrun/run-aspnetcore-microservices).

In this forked version, you will find the following features:

# Features
## Deployment 

I created the deployment code using:

- Deployment to a local Kubernetes instance (**Minikube**), using **Helm charts**.
- Installation of **Istio** as a service Mesh solution.
- Using **Lens** for cluster management.
- Using **Ingress controller** to expose the application from the outside Kubernetes cluster.

## Observability 

The following tools are availble using this deployment code:

- **Elasticsearch** and **Kibana**: Kibana is a data visualization and exploration tool used for log and time-series analytics and application monitoring. It uses Elasticsearch as search engine.
- **Healthchecks** implemented in each microservices using **AspNet Core health checks features**.
- **Kiali** : observability console for Istio with service mesh configuration and validation capabilities. It helps you understand the structure and health of your service mesh by monitoring traffic flow to infer the topology and report errors.
- **Jaeger** : open source software for tracing transactions between distributed services. It's used for monitoring and troubleshooting complex microservices environments.
- **Prometheus** and **Grafana**: Prometheus is free and an open-source event monitoring tool for containers or microservices. Grafana is a multi-platform visualization software available since 2014.

## Scaling (To be done)
- **HPA**
- **Keda**

# Run the application using Docker

In order to run the application on the local machine, follow the [original repository documentation](https://github.com/aspnetrun/run-aspnetcore-microservices).

# Run the application using Local Kubernetes
## Create your local Kubernetes Cluster

Minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes. Follow the installation documentation below:

[Minikube offical documentaion](https://minikube.sigs.k8s.io/docs/start/)

After the installation is finished with success, you should be able to see the Minikube pods running like this:

![minikube](/doc/minikube_1.png)

## Create local Registry

A container registry is a repository, or collection of repositories, used to store container images for Kubernetes, DevOps, and container-based application development.

I decided to create my own local container registry, but you can use whatever you want to host your container images. 

I followed this [documentation](https://minikube.sigs.k8s.io/docs/handbook/registry/) to create my Registry using Minikube.

Once you have the addon enabled, you should be able to connect to it. When enabled, the registry addon exposes its **port 5000** on the minikube’s virtual machine.

On your local machine you should now be able to reach the minikube registry by doing "port foward":

``kubectl port-forward --namespace kube-system service/registry 5000:80``

and run the curl below:

``curl http://localhost:5000/v2/_catalog``

## Installing Helm

You will need to install Helm locally to be able to run the deployment script available in this repo.

Please see the official documentation [here](https://helm.sh/docs/intro/install/).

## Installing Lens

Lens is a a Kubernetes IDE — open source project. Available for Linux, Mac, and Windows, Lens gives you a powerful interface and toolkit for managing, visualizing, and interacting with multiple Kubernetes clusters.

It will make your life easier, but you will also start forgeting all the kubectl commands you used to use.

The installation link is [here](https://www.mirantis.com/software/lens/).

## Application Deployment

Assuming that you already have your container images built, you should now push them to the registry. To do that, you should (example):

``kubectl port-forward --namespace kube-system service/registry 5000:80``

Tag the image:

``docker tag ocelotapigw localhost:5000/ocelotapigw``

Push to the registry:

``docker push localhost:5000/ocelotapigw``

Verify if the images are available on the registry:
![registry](/doc/registry_images.png)


After that, you are ready to start the application deployment into the Kubernetes cluster. The next step is to run the script below, that will create the pods, services and other K8S resources needed to run the application.

### Installing the application using helm charts

You will need Powershell installed. If you are using Linux like me, take a look on this [link](https://adamtheautomator.com/powershell-linux/).

Go to the folder /run-aspnetcore-microservices/deployment/k8s/helm and run:

``pwsh``

![pwsh](/doc/pwsh.png)

Run the script below:

``./deploy-all.ps1``

![run_deploy](/doc/run_deploy.png)

You should see the pods running after some seconds:

![pods_running](/doc/pods_running.png)

## Accesing the application

At this point, the application should be available. You can access using one of the following options:

### Option (1): node port

Throught the node port **8089** configured on the file **deployment/k8s/helm/aspnetrunbasics/values.yaml**.

You can do a port forward to web application service exposed on this 8089 port: 

``kubectl port-forward --namespace default service/aspnetrun-aspnetrunbasics [YOUR_LOCAL_PORT]:8089``

### Option (2): using Lens

If you are using Lens, go to PODS, click on the **aspnetrunbasics** POD and click on **Ports** link:

![run_deploy](/doc/lens_aspnet.png)

### Option (3): cluster IP and service port

You can access using your **[cluster IP]:[Service Port]** exposed by the Web application service. To identify the cluster IP, you can you use:

``minikube Ip`` or ``kubectl cluster-info``

In my case, my cluster IP is 192.168.49.2:

![run_deploy](/doc/cluster_ip.png)

To identify the web application service port, you can you use:

``kubectl get svc | grep aspnetrunbasics``

In my case, my service port is 31293:

![run_deploy](/doc/service_port.png)

And, using the browser:

![run_deploy](/doc/browse_app.png)

## Accessing the application Web Status

You can follow the same options (1 and 2) explained above, but accessing the **webstatus** POD. The option (3) is not available, because this POD is not available outside the cluster.

![run_deploy](/doc/webstatus.png)

![run_deploy](/doc/webstatusbrowser.png)

## Accessing the APIs using Swagger

The microservices APIs are only available within the cluster. You can also use port-forward or access via LENS (/swagger/).

* catalog
* basket
* discount
* ordering

![run_deploy](/doc/swagger.png)


## Accessing Kibana (Elasticsearch)

The Kibana is only accessible within the cluster. You can also use port-forward or access via LENS. In the first access, you will need to configure the elasticsearch Index to be able to see the application logs. The configuration is beyond the scope of this documentation, but all the microservices are configure to send logs to the Elasticsearch container also running in the cluster.

![run_deploy](/doc/kibana_lens.png)

![run_deploy](/doc/kibana.png)

# Using Istio

Istio manages traffic flows between services, enforces access policies, and aggregates telemetry data, all without requiring changes to application code.

## Installing Istio in the Minikube Cluster

The configuration files below will generate the resources (pods, services, service accounts, CRD, etc) needed to install Istio on your Minikube cluster:

``kubectl apply -f 1-istio-init.yaml``

``kubectl apply -f 2-istio-minikube.yaml``

``kubectl apply -f 3-kiali-secret.yaml``

It will also install **Kiali**, **Prometheus**, **Grafana** and **Jaeger**.

After you run it, you should see the containers running in the **istio-system** namespace:

``kubectl get po -n istio-system``

![istio_pods](/doc/istio_pods.png)

## Enabling Istio

Once you have Istio installed, you can enable it. To do that, we will create a **label** on the namespace used by the application (`Default` in this case). 

``kubectl label namespace default istio-injection=enabled``

To confirm the label creation:

``kubectl describe ns default``

This label will be used to determine whether Istio should be injected on the desired containers. By default, all the containers will be using Istio, except the ones explicitly configured to not use it. This configuration is done throught the deployment.yaml. For example, we will not inject Istio on Mongodb container, as shown below:

![istio_inject_false](/doc/istio_inject_false.png)

In order to inject Istio in the application, you need to redeploy it.

Go to the folder /run-aspnetcore-microservices/deployment/k8s/helm and run the **Powershell** script:

``./deploy-all.ps1``

Now you should see the **Sidecar** containers injected in some of the Pods:

![istio_proxy](/doc/istio_proxy.png)

## Accessing Kiali, Grafana and Jaeger

These tools are only accessible within the cluster. You can either use port-forward or access via LENS:

### Kiali

![tools](/doc/kiali_1.png)

![tools](/doc/kiali_2.png)

![tools](/doc/kiali_3.png)

![tools](/doc/kiali_4.png)

### Grafana and Prometheus

![tools](/doc/grafana_1.png)

![tools](/doc/grafana_2.png)

![tools](/doc/grafana_3.png)

![tools](/doc/grafana_4.png)

### Jaeger

This tool is running on container [Cluster IP] / port 31001:

![tools](/doc/Jaeger_1.png)

![tools](/doc/Jaeger_2.png)


## Using Nginx Ingress controller to expose the application

So far we have exposed the web application using a **Node port** defined on the `aspnetrunbasics` helm chart (values.yaml). Using this configuration, Kubernetes will allocate a specific port on each Node to the web application service, and any request to your cluster on that port will be forwarded to the service.

It works, but there is a more decouple/better way to implement it. See this documentation [here](https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html).

### Enabling Ingress controller on your cluster

Run the following command on Minikube:

``minikube addons enable ingress``

You should see the Ingress controller pods running:

![ingress_1](/doc/ingress_1.png)

### Creating Nginx Ingress controller to the web app

Go to the folder: `deployment/k8s/ingress` and run the command:

``kubectl apply -f ingress.yaml``

Now you should see the Ingress created(*):

``kubectl get ingress -n default``

![ingress_2](/doc/ingress_2.png)

Note that I am using `my-minikube` as HOST name. You should configure your `/etc/hosts` file to be able to resolve this name to your Minikube cluster IP.

In my case, now I can access the web application using:

http://my-minikube/

The official Ingress documentation is [here](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/).

### Known issue

(*) If you receive this message when creating the ingress: **Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io"**, run the following work around (for studying purpose only):

``kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission``

# Versions used on this demo

* minikube 1.19.0
* Helm : 3.7.0
* ISTIO 1.10.3
