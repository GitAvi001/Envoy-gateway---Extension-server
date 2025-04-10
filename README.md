
# Envoy Gateway and Extension-server

This project implement a REST API invocation and sending the backend response process by extending envoy gateway using extension server.

Before sending the desired xDS configurations to the envoy proxy, envoy gateway can configured calling an external server over gRPC calls.

Below given diagram shows how this was done using envoy gateway and it's extension server.

[!Extension-server-architecture](images/Extension-server-architecture.png)

[!Extension-server-API-invoking-process](images/extension-server.png)

In this project I used Rancher desktop to use kubernetes and Docker. This can be done also using minikube.

## Prequisities

Before setting up the cluster and pods make ensure to install these Prequisities. 

1. Rancher desktop
2. Install Go
3. Install Visual Studio Code
3. Enable vs code extensions (YAML, kubernetes, Docker Extension Pack)
4. Start Rancher Desktop



## Steps - Setting up the pods

1. Install Gateway API CRDs and Envoy Gateway with creating the namespace envoy-gateway-system:

```bash

helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.1.4 -n envoy-gateway-system --create-namespace 
```
2. Make ensure to check pods are running without any issue:

```bash
kubectl get pods -n envoy-gateway-system  
```

3. clone the repo:
```bash
git clone https://github.com/GitAvi001/Envoy-gateway---Extension-server.git 
```

4. Move to the directory where Makefile is located:
```bash
cd /Envoy-gateway---Extension-server/examples/extension-server
```

5. Build the extension server and push the docker image to the docker hub:
(Make sure to correctly tage the image if you are using your own docker hub repository)

```bash
make image
```

6. Move to the directory where extension server's values.yaml file is located :

```bash
cd /Envoy-gateway---Extension-server/examples/extension-server/charts/extension-server
```
(update image.repository field as <dockerhub_username>/<repository_name>)

7. Make ensure to check pods are running without any issue:
```bash

kubectl get pods -n envoy-gateway-system  
```

8. Check the logs for each pod if any issues and fix them:
```bash

i. kubectl get pods -n envoy-gateway-system 

ii. kubectl logs -n envoy-gateway-system <pod_name>
```

## Steps - Apply custom resource definitions(CRD) 

a. Use Makefile to generate desired CRD

```bash 
cd ./examples/extension-server

i. make generate 

ii. make manifests
```
(CRD can generate with desired versions removing the commented lines in manifests commands)

## Steps - Apply cluster role and cluster role binding to the generated CRD
```bash
i. cd ./examples/extension-server/project

ii. kubectl apply apk-rbac.yaml -n envoy-gateway-system
```

## Steps - Apply custom resources
```bash
i. cd ./examples/extension-server/project/apk-cr

ii. kubectl apply -f .-n envoy-gateway-system
```

## Steps - Check pods and services 
```bash
i. kubectl get pods -n envoy-gateway-system

ii. kubectl get svc -n envoy-gateway-system
```
## Steps - Check API 
```bash
kubectl get apis -n envoy-gateway-system
```
## Steps - Check HTTPRoute and backend correctly working
a. Use Envoy gateway config dump to check whether configured HTTPRoutes applied successfully.

```bash
i. kubectl get pods -n envoy-gateway-system

ii. kubectl port-forward -n envoy-gateway-system <envoy_gateway_pod> 19000:19000
```
b. port-forward to the backend service and test the backend first directly.
```bash
i. kubectl get svc -n envoy-gateway-system

ii. kubectl port-forward -n envoy-gateway-system svc/<envoy_gateway_service> 8080:80
```
c. Import this curl command to postman and send the request.
```bash
curl -v -H "Host: www.example.com" "http://localhost:8080/apk-http-route/api/v3/pet/findByStatus?status=available"
```

## Steps - Edit server.go and redeploy extension server
To extract the basePath or context path correctly from the applied API CR and map to the correct backend resource server.go needed to edit correctly.

After editing the server.go correctly extension-server should redpoly to the kubernetes cluster.

For testing purpose server.go already modified to extract basePath and HTTPRoute from applied CR dynamically.

```bash
i. cd ./examples/extension-server

ii. ./deploy-server.sh
```

## Steps - Invoke API
a. port-forward to envoy-gateway-service
```bash
i. kubectl get svc -n envoy-gateway-system

ii. kubectl port-forward -n envoy-gateway-system svc/<envoy_gateway_service> 8080:80
```
b. Import this curl command to postman and invoke the API. 
```bash
curl -v -H "Host: www.example.com" "http://localhost:8080/my-api/apk-http-route/api/v3/pet/findByStatus?status=available"
```

## Steps- error checking 

Check if any errors in the logs.

```bash
i. kubectl logs -n envoy-gateway-system <envoy_gateway_pod>

ii. kubectl logs -n envoy-gateway-system <extension_server_pod>
```

## 🔗 Links

Rancher Desktop installation: https://docs.rancherdesktop.io/getting-started/installation/

Go(version 1.23.3) installation: https://go.dev/doc/install

Visual studio Code installation: https://code.visualstudio.com/download

Envoy Gateway: https://gateway.envoyproxy.io/

Envoy Gateway Extension Server: https://gateway.envoyproxy.io/v1.1/tasks/extensibility/extension-server/










