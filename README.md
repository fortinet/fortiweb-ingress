

# Fortiweb Ingress Controller
Below content is the basic know-how and quick start for FortiWEB Ingress Controller.
For more much details, please refer to the [official document](https://docs.fortinet.com/document/fortiweb/7.4.0/ingress-controller-installation-guide/742835/fortiweb-ingress-controller-overview).

</br>
</br>
</br>

![FortiWEB Ingress Controller Overview](https://github.com/fortinet/fortiweb/ingress-controller/blob/main/figures/fwb-ingress-controller-overview.png?raw=true)

The FortiWEB Ingress Controller fulfills the Kubernetes Ingress resources and allows you to manage FortiWEB objects from Kubernetes. It is deployed in a container of a pod in a Kubernetes cluster. The list below outlines the major functionalities of the FortiWEB Ingress Controller: 

 - To list and watch Ingress related resources, such as Ingress, Service, Node and Secret. 
 - To convert Ingress related resources to FortiWEB objects, such as virtual server, content routing, real server pool, and more.
 - To handle Add/Update/Delete events for watched Ingress resources and automatically implement corresponding actions on FortiWEB.
 
 
 ![Ingress](https://github.com/fortinet/fortiweb/ingress-controller/blob/main/figures/ingress.png?raw=true)

Ingress is a Kubernetes object that manages the external access to services in a Kubernetes cluster (typically HTTP/HTTPS). Ingress may provide load-balancing and name-based virtual hosting.

The FortiWEB Ingress Controller combines the capabilities of an Ingress resource with the Ingress-managed load balancer, FortiWEB. 

FortiWEB, as the Ingress-managed load balancer, not only provides flexibility in load-balancing, but also guarantees more security with features such as the Web Application Firewall (WAF), machine leaning to protect the web server resources in the Kubernetes cluster. Other features such as health check, persistence.

## Supported Release and Version

<table>
    <thead>
        <tr>
            <th>Product</th>
            <th colspan=4>Version</th>   
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>FortiWEB Ingress Controller</td>
            <td>1.0.0</td>
            <td>1.0.1</td>
            <td>1.0.2</td>
            <td>2.0.0</td>
        </tr>
        <tr>
            <td>Kubernetes</td>
            <td>1.19.8-1.23.x</td>
            <td>1.19.8-1.24.x</td>
            <td colspan=2>1.19.8-1.27.x</td>
        </tr>
        <tr>
            <td>FortiWEB</td>
            <td colspan=4>5.4.5 - 7.4.x*</td>
        </tr>
	    <tr>
            <td>Openshift Container platform</td>
            <td colspan=3>Not supported</td>
            <td> 4.7-4.12.x</td>
        </tr>
    </tbody>
</table>

>**Note** 
>Some features for FortiWEB Ingress Controller version >= 2.0.0 require FortiWEB version >= 7.4.0 to support. Please check the [release notes](https://github.com/fortinet/fortiadc-ingress/blob/main/Release-Notes.md).
## Supported Environment
The FortiWEB Ingress Controller has been verified to run in the Openshift Cluster in Openshift Container Platform environment and Kubernetes cluster in the below environments:
| Environment | Tools for Building |
|--|--|
| Private Cloud | kubeadm, minikube, microk8s |
| Public Cloud | AWS EKS, Oracle OKE |

## The Kubernetes API Version

To ensure you use an API version of Kubernetes objects that the FortiWEB Ingress Controller supports, you can use the kubectl command to check the resource API version.


    for kind in `kubectl api-resources | tail +2 | awk '{ print $1 }'`; do kubectl explain $kind; done | grep -e "KIND:" -e "VERSION:"

| API Object | API Version |
|--|--|
| Node | v1 |
| Pod | v1 |
| PodTemplate | v1 |
| ServiceAccount | v1 |
| Deployment | apps/v1 |
| ReplicaSet | apps/v1 |
| Endpoints | v1 |
| Event | v1 |
|IngressClass  | networking.k8s.io/v1 |
|Ingress  | networking.k8s.io/v1 |
|ClusterRoleBinding  | rbac.authorization.k8s.io/v1 |
|ClusterRole  | rbac.authorization.k8s.io/v1 |
|RoleBinding  | rbac.authorization.k8s.io/v1 |
|Role  | rbac.authorization.k8s.io/v1 |

# Installation
Install the FortiWEB Ingress Controller using Helm Charts.

:bulb: Currently, only Helm 3 (version 3.6.3 or later) is supported.

Helm Charts ease the installation of the FortiWEB Ingress Controller in the Kubernetes cluster. By using the Helm 3 installation tool, most of the Kubernetes objects required for the FortiWEB Ingress Controller can be deployed in one simple command. 

The Kubernetes objects required for the FortiWEB Ingress Controller are listed below:

|Kubernetes object| Description |
|--|--|
| Deployment | By configuring the replica and pod template in the Kubernetes deployment, the deployment ensures the FortiWEB Ingress Controller provides a non-terminated service. |
| Service Account | The service account is used in the FortiWEB Ingress Controller. |
| Cluster Role | A cluster role defines the permission on the Kubernetes cluster-scoped Ingress-related objects |
| Cluster Role Binding |The cluster role is bound to the service account used for the FortiWEB Ingress Controller, allowing the FortiWEB Ingress Controller to access and operate the Kubernetes cluster-scoped Ingress-related objects. |
| Ingress Class |The IngressClass "fwb-ingress-controller" is created for the FortiWEB Ingress Controller to identify the Ingress resource. If the Ingress is defined with the IngressClass "fwb-ingress-controller", the FortiWEB Ingress Controller will manage this Ingress resource. |

To get the verbose output, add --debug option for all the Helm commands.

## Get Repo Information
   

    helm repo add fortiweb-ingress-controller https://fortinet.github.io/fortiweb/ingress-controller/

    helm repo update

## Install FortiWEB Ingress Controller

    helm install first-release --namespace fortiweb-ingress --create-namespace --wait fortiweb-ingress-controller/fwb-k8s-ctrl

## Check the installation

    helm history -n fortiweb-ingress first-release
    
    kubectl get -n fortiweb-ingress deployments
    
    kubectl get -n fortiweb-ingress pods

Check the log of the FortiWEB Ingress Controller.

    kubectl logs -n fortiweb-ingress -f [pod name]
 
## Upgrading chart

    helm repo update
    helm upgrade --reset-values -n fortiweb-ingress first-release fortiweb-ingress-controller/fwb-k8s-ctrl

## Uninstall Chart

    helm uninstall -n fortiweb-ingress first-release

# Configuration parameters
## FortiWEB Authentication Secret

As shown in above figure, the FortiWEB Ingress Controller satisfies an Ingress by FortiWEB REST API call, so the authentication parameters of the FortiWEB must be known to the FortiWEB Ingress Controller.

To preserve the authentication securely on the Kubernetes cluster, you can save it with the Kubernetes secret. For example

    kubectl create secret generic fad-login -n [namespace] --from-literal=username=admin --from-literal=password=[admin password]

The secret is named fad-login. This value will be specified in the Ingress annotation "fortiweb-login" for the FortiWEB Ingress Controller to get permission access on the FortiWEB.

:warning:  The namespace of the authentication secret must be the same as the Ingress which references this authentication secret.

## Annotation in Ingress
Configuration parameters are required to be specified in the Ingress annotation to enable the FortiWEB Ingress Controller to determine how to deploy the Ingress resource.

|Parameter  | Description | Default |
|--|--|--|
| fortiweb-ip | The Ingress will be deployed on the FortiWEB with the given IP address. <br> **Note**: This parameter is **required**. | |
| fortiweb-login | The Kubernetes secret name preserves the FortiWEB authentication information. <br> **Note**: This parameter is **required**. | |
| fortiweb-ctrl-log | Enable/disable the FortiWEB Ingress Controller log. Once enabled, the FortiWEB Ingress Controller will print the verbose log the next time the Ingress is updated. |enable |
| virtual-server-ip | The virtual server IP of the virtual server to be configured on the FortiWEB. This IP will be used as the address of the Ingress. <br> **Note**: This parameter is **required**. | |
| virtual-server-interface | The FortiWEB network interface for the client to access the virtual server. <br> **Note**: This parameter is **required**. | |
| virtual-server-addr-type | IP address type of virtual server. ||
| server-policy-web-protection-profile | Specify the predefined or user-defined web protection profile name. | |
| server-policy-https-service| Specify the predefined or user-defined https service name.|HTTPS |
| server-policy-http-service | Specify the predefined or user-defined http service name. | HTTP |
| server-policy-syn-cookie | Enable/Disable syn cookie to prevent SYN floods. | disable |
| server-policy-http-to-https | Enable/Disable HTTP to HTTPS redirection. | disable|
| machine-learning-api-ip-list-type | Block/trust the ip address list.| trust |
| machine-learning-api-add-ip | The IP adress to include/exclude for collect data.| |
| machine-learning-api-del-ip  | he IP address remove from IP list. | |
| machine-learning-api-add-domain | The system will collect sample and build model for the domin. | |
| machine-learning-api-del-domain | The Domain name remove from domain list. | |
| machine-learning-bot-add-ip | Add IPS to allow FWB to collect sample data.| |
| machine-learning-bot-del-ip | Remove IPS to allow FWB to collect sample data.| |
| machine-learning-anomaly-add-domain | Add the domain to be protected by Anomaly detection. |  |
| machine-learning-anomaly-add-ip | The IP adress to include/exclude for collect data. | |
| machine-learning-anomaly-ip-list-type | Block/trust the ip address list. |trust|
| machine-learning-anomaly-del-ip | The IP address remove from anomaly IP list. | |
| machine-learning-anomaly-del-domain-list | Remove the domain to be protected by Anomaly detection. | |

## Annotation in Service

>**Warning**
>The FortiWEB Ingress Controller version 1.0.x only supports services of type **NodePort**.

|Parameter  | Description | Default |
|--|--|--|
| health-check-ctrl | Health check method used in server pool. |  |
| lb-algo | Load Balancing Algorithm used in server pool .| round-robin |
| persistence | Persistence Method used in server pool. ||

# Deployment of a Simple-fanout Ingress Example

![Simple-fanout example](https://github.com/fortinet/fortiweb/ingress-controller/blob/main/figures/simple-fanout.png?raw=true)

In this example, the client can access service1 with the URL https://test.com/info and access service2 with the
URL https://test.com/hello.
Service1 defines a logical set of Pods with the label run=sise. Sise is a simple HTTP web server.
Service2 defines a logical set of Pods with the label run=nginx-demo. Nginx is also a simple HTTP web server.
Services are deployed under the namespace default.

## Deploy the Pods and expose the Services

Service1:

    kubectl apply -f https://raw.githubusercontent.com/fortinet/fortiweb/ingress-controller/main/service_examples/service1.yaml
Service2:

    kubectl apply -f https://raw.githubusercontent.com/fortinet/fortiweb/ingress-controller/main/service_examples/service2.yaml

## Deploy the Ingress

Download the simple-fanout-example.yaml


    curl -k https://raw.githubusercontent.com/fortinet/fortiweb-ingress/main/ingress_examples/simple-fanout-example.yaml -o simple-fanout-example.yaml

Modify the Ingress Annotation in simple-fanout-example.yaml to accommodate to your environment, ex: fortiadc-ip, virtual-server-ip, etc.. Then deploy the ingress with kubectl command

    kubectl apply -f simple-fanout-example.yaml


Try to access https://test.com/info.

![test_info](https://github.com/fortinet/fortiweb-ingress/blob/main/figures/test_info.png?raw=true)

Try to access https://test.com/hello.

![nginx-demo](https://github.com/fortinet/fortiweb-ingress/blob/main/figures/nginx-demo.png?raw=true)

