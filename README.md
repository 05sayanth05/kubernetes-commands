# kubernetes-commands

## kubectl

### Get all nodes
```
kubectl get nodes
```

### Pods

#### Get all pods
```
kubectl get pod
```

#### More info of pods
```
kubectl get pod -o wide
```

#### Describe a pod
```
kubectl describe pod <name_of_pod>
```

### Deployment

#### Get all deployment
```
kubectl get deployment
```

#### Get replica set
```
kubectl get replicaset
```

#### Create
```
kubectl create deployment <name_of_deployment> --image=<image_name> -- [COMMAND] [args...] [options]
```

- The image will be downloaded (default) from docker hub

#### Edit
```
kubectl edit deployment <name_of_deployment>
```

#### Delete
```
kubectl delete deployment <name_of_deployment>
```

- This will delete the deployment as well as the pods of this deployment

#### Applying deployment file
```
kubectl apply -f <file_name>
```

#### Describe a deployment
```
kubectl describe deployment <deployment_name>
```

#### Deployment configuration file syntax (yaml file)
```
apiVersion: <version>
kind: Deployment

metadata:
  name: <name_of_deployment>
  labels:
    <key>: <value>

spec:
  replicas: <number_of_replicas_needed>
  selector:
    matchLabels:
      <key>: <value>
  template:
    metadata:
      labels:
        <key>: <value>
    spec:
      containers:
      - name: <name_of_image>
        image: <image_name:version>
        ports:
        - containerPort: <port>
```

- The file has mainly three parts,
  - metadata
  - specification
  - status (managed by kubernetes, kubernetes will always check desired and actual state, if they are different then kubernetes will try to fix it. this is the basis of self healing feature. the data of the status is from etcd)
- The things under template is applied for a pod
- Label and selector defines the connection between services and deployments (its pods also)
- The selector under spec matches all the pods with the corresponding label to this deployment

### Service

#### Syntax of service file for deployment
```
apiVersion: <version>
kind: Service

metadata:
  name: <service_name>

spec:
  selector:
    <key>: <value>
  ports:
    - protocol: TCP
      port: <port_in_which_the_service_should_listen_for_requests>
      targetPort: <port_in_which_the_pod_is_listening_that_is_containerPort>
```

- The request arrives at the port in which the service is listening
- This request have to be forwarded to a pod which also has a port in which the application is running, that port is the targetPort
- The targetPort have to be same as containerPort defined in the deployment 

#### Get services
```
kubectl get service
```

#### Describe a service
```
kubectl describe service <service_name>
```

### Debugging

#### Logs of a pod
```
kubectl logs <name_of_pod>
```

#### Get into the terminal of a pod
```
kubectl exec -it <name_of_pod> -- bin/bash
```

- `it` is interactive terminal
