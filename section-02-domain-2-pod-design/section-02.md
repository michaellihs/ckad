# Section 2 - POD Design

## Labels

### Create a POD with a label

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mywebserver
  labels:
    env: dev
```

### Filter PODs by labels

```bash
kubectl get pods -l env=dev
```

### Show labels of PODs

```bash
kubectl get pods --show-labels
```

### Add labels to existing PODs

```bash
kubectl label pod mywebserver tier=frontend
```

### Remove labels from PODs

```bash
# mind the '-' after the key of the label
kubectl label pod mywebserver tier-
```

## Replicasets

### Basic config

```yaml
# mind the different namespace 'app'
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: kplabs-replicaset
spec:
  # how many replicas do you want
  replicas: 3
  # for the replicaset to figure out how many pods are running
  selector:
    matchLabels:
      tier: frontend
  # template resembles the configuration of a pod (without the 'apiVersion' and kind 'fields')
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx
```

## Deployments

- Deployments sit on the top of ReplicaSets
- Used to roll out new version of our application
- Allows for rollback in case of a rollout of a new version fails
- Ensures that only a certain number of PODs are down, while upgrading to a new version

### Get roll out history

```bash
kubectl rollout history deployment.v1.apps/kplabs-deployment
```

### Create a Deployment

```bash
kubectl apply -f yamls/deployment.yaml
```

There is a replicaset created for this deployment:

```bash
kubectl get replicaset

# NAME                           DESIRED   CURRENT   READY   AGE
# kplabs-deployment-5c4db59958   3         3         3       74s
```

### Update a Deployment

```bash
kubectl apply -f yamls/deployment-2.yaml
```

This returns in a new replicaset for the updated deployment:

```text
$ kubectl get replicasets  
NAME                           DESIRED   CURRENT   READY   AGE
kplabs-deployment-5c4db59958   0         0         0       3m54s
kplabs-deployment-6849f7884b   3         3         3       14s
```

To get an overview of how roll out happened, use `kubectl describe`

```bash
kubectl describe deployment kplabs-deployment
```

To get an overview of the rollout history of the deployment, use `kubectl rollout history`

```text
$ kubectl rollout history deployment.v1.apps/kplabs-deployment  
deployment.apps/kplabs-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

To get further information of a revision, provide the `--revision` parameter:

```text
$ kubectl rollout history deployment.v1.apps/kplabs-deployment --revision 1
deployment.apps/kplabs-deployment with revision #1
Pod Template:
  Labels:       pod-template-hash=5c4db59958
        tier=frontend
  Containers:
   nginx:
    Image:      nginx
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
  Node-Selectors:       <none>
  Tolerations:  <none>
```

### Roll back a Deployment

```bash
kubectl rollout undo deployment.v1.apps/kplabs-deployment --to-revision=1
```

### Create Deployments via CLI

```bash
kubectl create deployment my-deployment --image=nginx --replicas=3 --dry-run=client -o yaml
```

You can get further examples for creating deployments with the CLI via:

```bash
kubectl create deployment --help
```

### maxSurge and maxUnavailable

- `maxSurge` maximum number of pods that can be scheduled above original number of pods
- `maxUnavailable` maximum number of pods that can be unavailable during an update

These values can be defined both in `%` and absolute numbers.

### Set an image for a Deployment

```bash
kubectl set image deployment kplabs-deployment nginx=httpd
```

where `nginx` refers to the name of the container for which you want to change the image.

### Scale number of replicas for a Deployment

```bash
kubectl scale deployment kplabs-deployment --replicas 10
```

## Jobs

### Run a job

Create a file:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: kplabs-job
spec:
  template:
    spec:
      containers:
        - name: kplabs-pods
          image: busybox
          command: ["/bin/sh"]
          args: ["-c", "echo Hello World"]
      restartPolicy: Never
```

apply the file:

```bash
kubectl apply -f yamls/job.yaml
```

check that the container has run via:

```bash
kubectl get pods
```

check the log output via

```bash
kubectl logs <POD NAME>
```

check the job status via

```bash
kubectl get jobs

# NAME         STATUS     COMPLETIONS   DURATION   AGE
# kplabs-job   Complete   1/1           4s         2m7s
```

### Delete a Job

```bash
kubectl delete job kplabs-job
```

## Cron Jobs

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: kplabs-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: kplabs-pods
              image: busybox
              args: [ "/bin/sh", "-c", "date; echo Hello from the Kubernetes cluster" ]
          restartPolicy: OnFailure
```

apply the file via

```bash
kubectl apply -f yamls/cronJob.yaml
```

after a while you should see a job via

```bash
kubectl get jobs
```

that creates a pod

```bash
kubectl get pods
```

and you can see the logs via

```bash
kubectl logs <POD NAME FROM ABOVE>
```
