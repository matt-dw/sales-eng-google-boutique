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
- **Synthetic Load Generation:** The application demo comes with a background
  job that creates realistic usage patterns on the website using
  [Locust](https://locust.io/) load generator.

## Installation

For all installation options, see [the project's original README.md](https://github.com/GoogleCloudPlatform/microservices-demo)

### Prerequisites

   - kubectl (can be installed via `gcloud components install kubectl`)
   - [edgectl](https://www.getambassador.io/docs/latest/tutorials/getting-started/)
   - [skaffold]( https://skaffold.dev/docs/install/) ([ensure version ≥v1.10](https://github.com/GoogleContainerTools/skaffold/releases))

### Running on Google Kubernetes Engine (GKE)

1.  Create a Google Kubernetes Engine cluster and make sure `kubectl` is pointing
    to the cluster.

    ```sh
    k gke create --node-count 4
    ```

2.  Install Ambassador Edge Stack, with Traffic-Agent RBAC in the `default` namespace.

    ```sh
    edgectl install
    k apply -f https://getambassador.io/yaml/traffic-agent-rbac.yaml
    ```

3.  Deploy the demo code
    1. Using Pre-Built Container Images:
    
        Run `k apply -n default -f ./release/kubernetes-manifests.yaml` to deploy the app.
        
    2. Building images from scratch:
    
        In the root of this repository, run `skaffold run --default-repo=`,
        where [PUBLIC_REPO] is your Docker Hub repository or GCR `gcr.io/[PROJECT_ID]`.
    
        This command:
    
        - builds the container images
        - pushes them to the container registry
        - applies the `./kubernetes-manifests` deploying the application to
          Kubernetes.

### Using Service Preview

See the Service Preview Quick Start for more information.

The `frontend`, `currencyservice` and `adservice` are already instrumented with annotation and automatically injects the Service Preview Traffic-Agent.

You should be able to connect to the remote cluster:

    ```sh
    sudo edgectl daemon
    edgectl connect
    ```
    
Check the status of the connection and the available intercepts:

    ```sh
    edgectl status
    edgectl intercept list
    edgectl intercept avail
    ```
    
Intercept the `frontend` deployment and the `currencyservice` deployment, assuming they are running on your local workstation:

    ```sh
    edgectl intercept add frontend -n my-frontend-intercept -t localhost:8080
    ```
    
  The local `frontend` service is now accessible using the given Preview URL.
  Re-use the same UUID token to intercept requests going to the GRPC `currencyservice` as headers are propagated from the `frontend` service to its dependencies:
    
    ```sh
    edgectl intercept add currencyservice -n my-currencyservice-intercept -m "x-service-preview=$UUID$" -t localhost:7000 --grpc
    ```