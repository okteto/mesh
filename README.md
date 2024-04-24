# Mesh

A sample demonstrating how to share a namespace among developers using Nginx as a Service Mesh.

## Problem Statement

As applications scale, managing a large number of microservices across multiple development namespaces becomes complex and resource-intensive. In such scenarios, deploying all microservices in each development namespace is impractical. This sample addresses this challenge by introducing a `shared` namespace concept within a Kubernetes cluster. Developers can deploy only the microservices they are actively working on in their individual namespaces while accessing other `shared` services from a common namespace. We achieve this using Nginx as a reverse proxy and [DNS Searches](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pod-dns-config).

## Application Scenario

Consider an application composed of numerous microservices, each exposing a public endpoint.
For this sample, we will use two microservices only for simplicity:

- [nginx-mash-backend-1](https://github.com/okteto/nginx-mesh-backend-1)
- [nginx-mash-backend-2](https://github.com/okteto/nginx-mesh-backend-2)

In our scenario, microservices communicate solely via public endpoints. If your setup differs, you'll need to configure DNS Searches across your microservices accordingly (more on this later).

We deploy an Nginx container containing all public hosts of our microservices. This deployment is outlined in [the Nginx Ingress file](kubernetes/ingress-template.yml).

Additionally, Nginx is configured to handle incoming requests and redirect them to the appropriate Kubernetes service using a configuration file like [default.conf](default.conf).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mesh
  labels:
    app: mesh
spec:
  template:
    spec:
      dnsConfig:
        searches:
          - ${OKTETO_NAMESPACE}.svc.cluster.local
          - ${OKTETO_SHARED_NAMESPACE}.svc.cluster.local
          - svc.cluster.local
          - cluster.local
```

This configuration instructs Nginx to prioritize service resolution within its namespace. If the service isn't found there, it defaults to the ${OKTETO_SHARED_NAMESPACE} namespace, which can be defined as an Okteto Variable representing your `shared` namespace.

As a result, Nginx redirects requests, such as mesh-backend-1-${OKTETO_NAMESPACE}.${OKTETO_DOMAIN}, to mesh-backend-1. If mesh-backend-1 is deployed in the current namespace, it resolves to this IP. Otherwise, it resolves to the IP of the mesh-backend-1 service in the ${OKTETO_SHARED_NAMESPACE} namespace.

## Internal Traffic

In scenarios where microservices communicate via the private cluster network, additional DNS configurations are required. Each microservice deployment must include DNS searches to facilitate internal communication:

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      dnsConfig:
        searches:
          - ${OKTETO_NAMESPACE}.svc.cluster.local
          - ${OKTETO_SHARED_NAMESPACE}.svc.cluster.local
          - svc.cluster.local
          - cluster.local
```

These DNS searches ensure that microservices prioritize service resolution within its namespace. And fallback to the `shared` namespace as needed.