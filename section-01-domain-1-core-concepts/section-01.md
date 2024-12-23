# Core Concepts

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