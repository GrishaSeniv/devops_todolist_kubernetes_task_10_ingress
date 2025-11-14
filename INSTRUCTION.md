## Setup and Deployment

The bootstrap.sh script contains all the commands to create the cluster and deploy the infrastructure.

Run the following commands to initialize the KinD cluster and deploy all resources.

```bash
kind create cluster --name todoapp-cluster --config cluster.yml

./bootstrap.sh
```

## Verify Resource Status

Check the status of all relevant resources, ensuring the Ingress pod is on the correct node and
the application is running.

```bash
# Check the status of the Ingress Controller pod
kubectl get pods -n ingress-nginx -o wide

# Check the status of the application resources
kubectl get all -n todoapp
```

Expected result:

The ingress-nginx-controller pod should be in the Running state on the todoapp-cluster-control-plane node.
The todoapp pod and service should be in the Running and Ready states, respectively.

## Application Access Validation (Ingress Rule Check)

The primary validation is ensuring the Ingress resource correctly routes traffic from the host
machine (localhost) to application, and that the regex path rule is working as intended.

1. Validate Root Access (/)
   Since Ingress is set to listen on host port 80, you can directly access it via http://localhost.

```bash
curl http://localhost/
```

Expected result:

You should receive the HTML/JSON response from todoapp application, confirming the base path
routing is successful.

2. Validate Path Capture and Rewrite

Since Ingress is configured with path: /(.*) and nginx.ingress.kubernetes.io/rewrite-target: /$1, requesting an
arbitrary path should still resolve to application's root (/).

```bash
# Test with a dummy path: /test/path
curl http://localhost/test/path
```

✅ Expected result: You should still receive the HTML/JSON response from todoapp application. If the application is
successfully returned, it confirms:

The ingress-nginx.kubernetes.io/use-regex: "true" annotation is working.

The path: /(.*) matches the request.

The ingress-nginx.kubernetes.io/rewrite-target: /$1 annotation successfully forwards the request to the application
backend's root path (/).

3. Validate No 404 Status Codes

Access http://localhost/ in browser. Inspect the Network tab in the browser's developer console.

✅ Expected result:

The main page request and all associated assets should return with a 200 OK status code. There should
be no requests failing with a 404 status code.

## 🧹 Cleanup

When finished, delete the entire KinD cluster to clean up all resources (pods, services, volumes, etc.).

```bash
kind delete cluster --name todoapp-cluster
```
