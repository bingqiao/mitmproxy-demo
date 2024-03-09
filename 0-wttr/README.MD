# WTTR with no proxy
A simple golang app is deployed via Docker and Kubernetes.

## WTTR
This simple golang app accepts a request like `http://localhost/?loc=London`, and makes call to weather report server with a request like `https://wttr.in/London?0`, then forwards the response back.

## Docker
Run the app:
```
docker-compose up
```
Test the app by:
```
curl "http://localhost:8080/?loc=Reading"
```
You should get something like below in terminal:
```
Weather report: Reading

     \  /       Partly cloudy
   _ /"".-.     +7(3) °C       
     \_(   ).   ← 11 km/h      
     /(___(__)  10 km          
                0.0 mm  
```
Ctrl+C to stop the docker compose.

N.B. you may need to run the following to have the containers created by docker compose removed so to avoid conflict later when running other examples.
```
docker-compose down
```

## Kubernetes
This project is tested in a local K8s cluster create by Kind.
Once you have Kind installed, run the following to create a new cluster where `k8s-dev` is the cluster name I chose:
```
kind create cluster -n k8s-dev
```
If you haven't run `docker-compose up`, run the following to build the docker image first:
```
docker buildx build -t wttr .
```
The docker image needs to be made availabe to the Kubernetes cluster. For a local Kind cluster, this is done by:
```
kind load docker-image wttr:latest --name k8s-dev
```
You can check if the image has been loaded correctly by (replace the ${CLUSTER_NAME} by the cluster name chosen):
```
docker exec -it ${CLUSTER_NAME}-control-plane crictl images
```
Deploy `wttr` app to local cluster `k8s-dev`:
```
kubectl apply -f deployment.yaml
```
Check the new pod is up and running without error:
```
kubectl get pods
```
Open up another terminal and set up port-forward so the cluster service `wttr-service` just deployed can be accessed from local host:
```
kubectl port-forward service/wttr-service 8080:8080
```
Test the app by:
```
curl "http://localhost:8080/?loc=Reading"
```
Delete the deployed service and pods:
```
kubectl delete -f deployment.yaml
```

This concludes the first part of this tutorial.