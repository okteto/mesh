# Mesh

A sample to share a namespace between developers using Nginx as a Service Mesh.

## Problem to solve

Architecture based on microservices can Let's consider an application with several microservices. Each microservice exposes a public endpoint.

## Application

Let's consider an application with several microservices. Each microservice exposes a public endpoint.
In this sample, we will consider two microservices:

- [nginx-mash-backend-1](https://github.com/okteto/nginx-mesh-backend-1)
- [nginx-mash-backend-2](https://github.com/okteto/nginx-mesh-backend-2)

Let's also assume the communication of microservices is always via the public endpoints.