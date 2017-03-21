# Front Proxy

This front proxy example includes one front Envoy pod and two service pods.
Each service pod contains a simple Flask apps that runs behind another Envoy
proxy.

![diagram](https://lyft.github.io/envoy/docs/_images/docker_compose_v0.1.svg)

## Instructions

This document assumes that you have a Kubernetes cluster ready and your local
`kubectl` has been properly configured to talk with the cluster.

1. Run the pods

The front proxy pod is specified in `frontend.yaml` and the service pods are
specified in `backend.yaml`. Also, to enable Istio auth, you need to install
Istio CA component, which is specified in `istio-ca.yaml`. You can start all
pods with a single command:
```
kubectl create -f frontend.yaml -f backend.yaml -f istio-ca.yaml
```

2. Fetch service IP
The `frontend.yaml` specifies a service object of type `LoadBalancer`, which
instructs Kubernetes to assign an external IP address that is accessible
publicly. You can find the external IP as follows
```
kubectl get -n frontend-ns svc/frontend-service
```
Notice that it may take a few minutes before the value of the IP is populated.

Let's save the external IP into an environment variable so we can use it later:
```
EXTERNAL_IP=$(kubectl get -n frontend-ns svc/frontend-service \
    -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

3. Send requests
You can now `curl` against the front Envoy proxy:
```
curl http://${EXTERNAL_IP}/service/1
curl http://${EXTERNAL_IP}/service/2
```
Notice that the front Envoy proxy can properly route your traffic to different
backends according to the path parameter.

## Clean up

You can now delete all created services/pods/deployments:
```
kubectl delete ns/{frontend-ns,backend-ns,istio-ca}
```

## Disclaimer

The example here is heavily borrowed from the document of [Envoy
proxy](https://lyft.github.io/envoy/docs/install/sandboxes.html#front-proxy)
and ported to Kubernetes environment.
