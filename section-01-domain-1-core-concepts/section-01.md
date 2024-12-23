# Section 1 - Core Concepts

## Local Kubernetes

### With Colima

Use [colima](https://github.com/abiosoft/colima) for a local Kubernetes setup:

```bash
colima start --kubernetes --network-address
```

You can then select the colima-provisioned cluster via `kubectx`.

### Bring up K8S Dashboard (UI) with Colima

Following the instruction on the [Kubernetes Dashboard install page](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

```bash
# Add kubernetes-dashboard repository
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
# Deploy a Helm Release named "kubernetes-dashboard" using the kubernetes-dashboard chart
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
# Port forward dashboard port to localhost
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
```

You can now access the dashboard via: [https://localhost:8443/#/login](https://localhost:8443/#/login)

When asked for a token, run the following commands to generate a token for the service account `default` in the `kubernetes-dashboard` namespace:

```bash
kubectl create serviceaccount k8sadmin -n kube-system
kubectl create clusterrolebinding k8sadmin --clusterrole=cluster-admin --serviceaccount=kube-system:k8sadmin
kubectl -n kube-system create token k8sadmin
```

Paste this token into the login UI.

## Kubernetes in the Cloud

### Create a K8S cluster on Digitalocean

- Use [promo link](https://cloud.digitalocean.com/registrations/new?refcode=f6fcd01aaffb) to get a 200$ credit for 60 days for a new account
- [Create a new K8S cluster](https://cloud.digitalocean.com/kubernetes/clusters/new)
  - Reduce the number of worker nodes to 1
  - Reduce to the lowest available Node Plan (currently 12$ / month)
  - Chose a clustername

### Install Digital Ocean CLI

- Install the `doctl` cli:

    ```bash
    brew install doctl
    ```

- [Create an API token](https://cloud.digitalocean.com/account/api/tokens/new)
  - full access for 1 year

- Use token for `doctl` authentication:

    ```bash
    doctl auth init --context <CONTEXT>
    ```

### Connect to Digital Ocean K8S cluster via CLI

```bash
doctl kubernetes clusters list
doctl kubernetes clusters kubeconfig save lihs-k8s --alias lihs-k8s
```

## PODs

### Starting a container in Kubernetes

```bash
kubectl run mywebserver --image=nginx
```

### Exec into a container

```bash
kubectl exec -it mywebserver -- bash
```

### Delete a POD

```bash
kubectl delete pod mywebserver
```

## Resources to learn about K8S objects

- K8S API [http://localhost:8001/api/v1](http://localhost:8001/api/v1)
- [K8S API documentation](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.27/)
- `kubectl explain pod.spec.containers`

## K8S objects from  Yaml

### Create Pod from yaml

From within the directory of this file:

```bash
kubectl apply -f yamls/nginx-pod.yaml
```

### Create a yaml with the `kubectl` CLI

```bash
kubectl run mywebserver --image=nginx --port=80 --dry-run=client -o yaml

# apiVersion: v1
# kind: Pod
# metadata:
#   creationTimestamp: null
#   labels:
#     run: mywebserver
#   name: mywebserver
# spec:
#   containers:
#   - image: nginx
#     name: mywebserver
#     ports:
#     - containerPort: 80
#     resources: {}
#   dnsPolicy: ClusterFirst
#   restartPolicy: Always
# status: {}
```

## Node affinity

In the pod specification:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mywebserver
  name: mywebserver
spec:
  nodeSelector:
    disk: ssd
  containers:
  - image: nginx
    name: nginx
    resources: {}
```

Add a label to a node via

```bash
kubectl label node pool-hmown489v-eyjmd disk=ssd
```
