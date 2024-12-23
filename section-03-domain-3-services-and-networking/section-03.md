# Section 3 - Services and Networking

## Services

A service is a gateway in front of a collection of Pods which provides a single IP address and DNS through which Pods can be accessed.

A service is a gateway that distributes the incoming traffic between its endpoints.

### Order of things to create a Service

1. Create backend Pods (endpoints)

    ```bash
    kubectl run backend-pod-1 --image=nginx
    kubectl run backend-pod-2 --image=nginx
    ```
   Get IP addresses of backend pods:

    ```text
    $ kubectl get pods -o wide
       NAME            READY   STATUS    RESTARTS   AGE     IP             NODE                   NOMINATED NODE   READINESS GATES
    backend-pod-1   1/1     Running   0          3m1s    10.108.0.100   pool-hmown489v-eyjmd   <none>           <none>
    backend-pod-2   1/1     Running   0          2m44s   10.108.0.104   pool-hmown489v-eyjmd   <none>           <none>
    ```

2. Create a frontend Pod

    ```bash
    kubectl run frontend-pod --image=ubuntu --command -- sleep 3600
    ```

   Test connection to the backend:

    ```bash
    kubectl exec -it frontend-pod -- bash
    apt-get update && apt-get -y install curl
    curl 10.108.0.104
    curl 10.108.0.100
    ```

3. Create a Service

    ```bash
    kubectl apply -f yamls/service.yaml
    kubectl get services
    curl <SERVICE IP>:8080
    ```

4. Associate endpoints with service

   > The IP addresses of the endpoints are the IP addresses of the PODs created for the backend.

    ```bash
    kubectl apply -f yamls/endpoing.yaml
    kubectl get services
    kubectl describe <SERVICE NAME>
    kubectl exec -it frontend-pod --bash
    curl <IP ADDRESS OF SERVICE>:8080
    ```

## Service Type - ClusterIP
