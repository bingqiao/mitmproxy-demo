# WTTR with reverse proxy
We'll set up a reverse proxy in front of wttr service in this project.

## WTTR
The exposed port of wttr has been changed to 18080 while 8080 is listened to by the reverse proxy. The intention here is so that adding a reverse proxy doesn't require a client to change its server endpoint.

## Docker
The most important change to `docker-compose.yaml` is adding a `mitmproxy-r` container.

Run the app:
```
docker-compose up
```
Notice there is a folder named `mitmproxy` created under `1-mitmproxy-regular`. This folder is volume mounted to mitmproxy container and is used by mitmproxy to store generated certificates.

## Tests
Open the web UI for `mitmproxy` in browser at `http://localhost:8081`.

Test 1, request to `localhost:18080`, success expected:
```
curl "http://localhost:18080/?loc=Reading"
```
This request goes directly to `wttr` app without proxy.

Test 2, request to `localhost:8080`, success expected:
```
curl "http://localhost:8080/?loc=Reading"
```
This request goes through reverse proxy first so observe it shows up in mitmproxy web UI.

Test 3: request to `localhost:8080` with SSL, failure expected:
```
curl "https://localhost:8080/?loc=Reading"
```
You likely saw this error message: `unable to get local issuer certificate`, which occured because the certificate issued by `mitmproxy` is not verified by curl.

Copy certificate from insde `mitmproxy-r` container:
```
docker cp mitmproxy-r:home/mitmproxy/.mitmproxy/mitmproxy-ca-cert.pem ./mitmproxy-ca-cert.pem
```
Run the following and expect success:
```
curl "https://localhost:8080/?loc=Reading" --cacert ./mitmproxy-ca-cert.pem
```
Observe this request shows up in mitmproxy web UI.

Ctrl+C to stop the docker compose. Then do the clean up:
```
docker-compose down
```

# Kubernetes
If you haven't run `docker-compose up`, run the following to build the docker image first:
```
docker buildx build -t wttr .
```
The docker image needs to be made availabe to the Kubernetes cluster. For a local Kind cluster, this is done by:
```
kind load docker-image wttr:latest --name k8s-dev
```
You can check if the image has been loaded correctly by:
```
docker exec -it k8s-dev-control-plane crictl images
```
Deploy `wttr-reverse` app and `mitmproxy-reverse` to local cluster `k8s-dev`:
```
kubectl apply -f deployment.yaml
```
Check the new pod is up and running without error:
```
kubectl get pods
```
Open up another terminal and set up port-forward so the wttr cluster service `wttr-service-reverse` just deployed can be accessed at port 18080 (mapped to wttr container) from local host:
```
kubectl port-forward service/wttr-service-reverse 18080:18080
```
Open up another terminal and set up port-forward so the wttr cluster service `wttr-service-reverse` just deployed can be accessed at port 8080 (mapped to mitmproxy-reverse container for reverse proxy) from local host:
```
kubectl port-forward service/wttr-service-reverse 8080:8080
```
Open up another terminal and set up port-forward so the wttr cluster service `wttr-service-reverse` just deployed can be accessed at prot 8081 (mapped to mitmproxy-reverse container for reverse proxy web UI) from local host:
```
kubectl port-forward service/wttr-service-reverse 8081:8081
```
## Tests
Open the web UI for `mitmproxy` in browser at `http://localhost:8081`.

Test 1, request to `localhost:18080`, success expected:
```
curl "http://localhost:18080/?loc=Reading"
```
This request goes directly to `wttr` app without proxy.

Test 2, request to `localhost:8080`, success expected:
```
curl "http://localhost:8080/?loc=Reading"
```
This request goes through reverse proxy first so observe it shows up in mitmproxy web UI.

Test 3: request to `localhost:8080` with SSL, failure expected:
```
curl "https://localhost:8080/?loc=Reading"
```
You likely saw this error message: `unable to get local issuer certificate`, which occured because the certificate issued by `mitmproxy` is not verified by curl.

Copy certificate from insde `mitmproxy-reverse` container:
```
kubectl cp -c mitmproxy-reverse wttr-reverse:root/.mitmproxy/mitmproxy-ca-cert.pem ./mitmproxy-ca-cert.pem
```
Run the following and expect success:
```
curl "https://localhost:8080/?loc=Reading" --cacert ./mitmproxy-ca-cert.pem
```
Observe this request shows up in mitmproxy web UI.

Ctrl+C to stop the all three port-forward termnimal then run the following to cleanup:
```
kubectl delete -f deployment.yaml
```

This concludes the second part of this tutorial.
