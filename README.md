<p align="center">
<img src="src/frontend/static/icons/Hipster_HeroLogoCyan.svg" width="300"/>
</p>



**Online Boutique** is a cloud-native microservices demo application.
Online Boutique consists of a 10-tier microservices application. The application is a
web-based e-commerce app where users can browse items,
add them to the cart, and purchase them.

**Google uses this application to demonstrate use of technologies like
Kubernetes/GKE, Istio, Stackdriver, gRPC and OpenCensus**. This application
works on any Kubernetes cluster (such as a local one), as well as Google
Kubernetes Engine. It’s **easy to deploy with little to no configuration**.

If you’re using this demo, please **★Star** this repository to show your interest!

## Screenshots

| Home Page                                                                                                         | Checkout Screen                                                                                                    |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [![Screenshot of store homepage](./docs/img/online-boutique-frontend-1.png)](./docs/img/online-boutique-frontend-1.png) | [![Screenshot of checkout screen](./docs/img/online-boutique-frontend-2.png)](./docs/img/online-boutique-frontend-2.png) |

## Service Architecture

**Online Boutique** is composed of many microservices written in different
languages that talk to each other over gRPC.

[![Architecture of
microservices](./docs/img/architecture-diagram.png)](./docs/img/architecture-diagram.png)

Find **Protocol Buffers Descriptions** at the [`./pb` directory](./pb).

| Service                                              | Language      | Description                                                                                                                       |
| ---------------------------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](./src/frontend)                           | Go            | Exposes an HTTP server to serve the website. Does not require signup/login and generates session IDs for all users automatically. |
| [cartservice](./src/cartservice)                     | C#            | Stores the items in the user's shopping cart in Redis and retrieves it.                                                           |
| [productcatalogservice](./src/productcatalogservice) | Go            | Provides the list of products from a JSON file and ability to search products and get individual products.                        |
| [currencyservice](./src/currencyservice)             | Node.js       | Converts one money amount to another currency. Uses real values fetched from European Central Bank. It's the highest QPS service. |
| [paymentservice](./src/paymentservice)               | Node.js       | Charges the given credit card info (mock) with the given amount and returns a transaction ID.                                     |
| [shippingservice](./src/shippingservice)             | Go            | Gives shipping cost estimates based on the shopping cart. Ships items to the given address (mock)                                 |
| [emailservice](./src/emailservice)                   | Python        | Sends users an order confirmation email (mock).                                                                                   |
| [checkoutservice](./src/checkoutservice)             | Go            | Retrieves user cart, prepares order and orchestrates the payment, shipping and the email notification.                            |
| [recommendationservice](./src/recommendationservice) | Python        | Recommends other products based on what's given in the cart.                                                                      |
| [adservice](./src/adservice)                         | Java          | Provides text ads based on given context words.                                                                                   |
| [loadgenerator](./src/loadgenerator)                 | Python/Locust | Continuously sends requests imitating realistic user shopping flows to the frontend.                                              |

## Features

- **[Kubernetes](https://kubernetes.io)/[GKE](https://cloud.google.com/kubernetes-engine/):**
  The app is designed to run on Kubernetes (both locally on "Docker for
  Desktop", as well as on the cloud with GKE).
- **[gRPC](https://grpc.io):** Microservices use a high volume of gRPC calls to
  communicate to each other.
- **[Istio](https://istio.io):** Application works on Istio service mesh.
- **[OpenCensus](https://opencensus.io/) Tracing:** Most services are
  instrumented using OpenCensus trace interceptors for gRPC/HTTP.
- **[Stackdriver APM](https://cloud.google.com/stackdriver/):** Many services
  are instrumented with **Profiling**, **Tracing** and **Debugging**. In
  addition to these, using Istio enables features like Request/Response
  **Metrics** and **Context Graph** out of the box. When it is running out of
  Google Cloud, this code path remains inactive.
- **[Skaffold](https://skaffold.dev):** Application
  is deployed to Kubernetes with a single command using Skaffold.

## Installation

For all installation options, see [the project's original README.md](https://github.com/GoogleCloudPlatform/microservices-demo)

### Prerequisites

   - kubectl (can be installed via `gcloud components install kubectl`)
   - [telepresence](https://www.getambassador.io/docs/telepresence/latest/quick-start/)
   - [skaffold]( https://skaffold.dev/docs/install/) ([ensure version ≥v1.10](https://github.com/GoogleContainerTools/skaffold/releases))
   
   - **Note: Telepresence and the preview URL functionality should work with most ingress configurations, including straightforward load balancer setups.  
Telepresence will discover/prompt during first use for this info and make its best guess at figuring this out and ask you to confirm or update   this.   
See the [Ambassador EdgeStack quick start guide](https://www.getambassador.io/docs/edge-stack/latest/tutorials/getting-started/) to deploy AES Ingress/API gateway if you are building a new cluser and need a ingress/gateway solution.

### Running on Google Kubernetes Engine (GKE)

1.  Create a Google Kubernetes Engine cluster with at least 4 nodes, and make sure `kubectl` is pointing to the cluster.

    ```sh
    # Using the Datawire kubectl-gke plugin
    kubectl gke create --node-count 4
    ```

2.  Deploy the demo code
    1. **Option 1** - Using Pre-Built Container Images:
    
        Deploy the Boutique app by running 
        
        ```sh
        kubectl apply -n default -f https://raw.githubusercontent.com/datawire/microservices-demo/master/release/kubernetes-manifests.yaml
        ```
        
    2. **Option 2** - Building images from scratch:
    
        In the root of this repository, run `skaffold run --default-repo=`,
        where [PUBLIC_REPO] is your Docker Hub repository or GCR `gcr.io/[PROJECT_ID]`.
    
        This command:
    
        - builds the container images
        - pushes them to the container registry
        - applies the `./kubernetes-manifests` deploying the application to
          Kubernetes.

### Using Telepresence

See the [Telepresence Quick Start](https://www.getambassador.io/docs/telepresence/latest/quick-start/) for more information.

The `frontend`, `currencyservice` and `adservice` are already instrumented.

You should be able to connect to the remote cluster:

```sh
telepresence connect
```
    
Check the status of the connection and the available intercepts:

```sh
telepresence status
telepresence list -n [namespace]
```
    
Intercept the `frontend` deployment and the `currencyservice` deployment, assuming they are running on your local workstation:

```sh
telepresence intercept frontend -n [namespace] --port 8080
```

You will be asked for the following information:

Ingress layer 3 address: This would usually be the internal address of your ingress controller in the format <service name>.namespace. For example, if you have a service ambassador-edge-stack in the ambassador namespace, you would enter ambassador-edge-stack.ambassador.

Ingress port: The port on which your ingress controller is listening (often 80 for non-TLS and 443 for TLS).

Ingress TLS encryption: Whether the ingress controller is expecting TLS communication on the specified port.

Ingress layer 5 hostname: If your ingress controller routes traffic based on a domain name (often using the Host HTTP header), this is the value you would need to enter here.

Telepresence supports any ingress controller, not just Ambassador Edge Stack.
For the example below, you will create a preview URL that will send traffic to the ambassador service in the ambassador namespace on port 443 using TLS encryption and setting the Host HTTP header to dev-environment.edgestack.me:

Terminal
```sh
$ telepresence intercept frontend -n boutique --port 8080

To create a preview URL, telepresence needs to know how cluster
ingress works for this service.  Please Confirm the ingress to use.

1/4: What's your ingress' layer 3 (IP) address?
     You may use an IP address or a DNS name (this is usually a
     "service.namespace" DNS name).

       [default: ambassador.ambassador]: 

2/4: What's your ingress' layer 4 address (TCP port number)?

       [default: 443]: 

3/4: Does that TCP port on your ingress use TLS (as opposed to cleartext)?

       [default: y]: 

4/4: If required by your ingress, specify a different layer 5 hostname
     (TLS-SNI, HTTP "Host" header) to access this service.

       [default: demo-aes.com]: 

Using Deployment frontend
intercepted
    Intercept name  : frontend-boutique
    State           : ACTIVE
    Workload kind   : Deployment
    Destination     : 127.0.0.1:8080
    Intercepting    : HTTP requests that match all of:
      header("x-telepresence-intercept-id") ~= regexp("e551e7c8-580a-4a0d-8401-e3643a2a8489:frontend-boutique")
    Preview URL     : https://heuristic-sutherland-2278.preview.edgestack.me
    Layer 5 Hostname: demo-aes.com

```    
The local `frontend` service is now accessible using the given Preview URL.  
Re-use the same UUID token to intercept requests going to the GRPC `currencyservice` as headers are propagated from the `frontend` service to its dependencies:  

Terminal    
```sh
telepresence intercept currencyservice -n boutique --http-match="x-telepresence-intercept-id"="e551e7c8-580a-4a0d-8401-e3643a2a8489:frontend-boutique" --port 7000 -u=false  
```

Note `-u=false` sets Preview URL to false for the currencyservice intercept.  Since we are using the same `--http-match=` we do not need to create a separate Preview URL.  
We will use the Preview URL created for the Frontend intercept.

With both intercepts running and both `frontend` and `currencyservice` running locally you can now make code changes to both or either and have those changes reflected in the Preview URL, without interfering with other developers.


To see this in action make the following changes to the frontend: 
../src/frontend/main.go file:

Line 40: defaultCurrency = "USD" , change to "BIT"
Line 55: "TRY": true}, append list to include "BIT":true}

```go
const (
	port            = "8080"
	defaultCurrency = "BIT"
	cookieMaxAge    = 60 * 60 * 48

	cookiePrefix    = "shop_"
	cookieSessionID = cookiePrefix + "session-id"
	cookieCurrency  = cookiePrefix + "currency"
)

var (
	whitelistedCurrencies = map[string]bool{
		"USD": true,
		"EUR": true,
		"CAD": true,
		"JPY": true,
		"GBP": true,
		"TRY": true,
		"BIT": true}
```

To see this in action make the following changes to the currencyservice: 
../src/currencyservice/data/currency_conversion.json:

Add a value for Bitcoin conversion.
```json
  "PHP": "59.083",
  "SGD": "1.5349",
  "THB": "36.012",
  "ZAR": "16.0583",
  "BIT": "0.002"
}
```

Save the files. Restart local services and reload your preview url.