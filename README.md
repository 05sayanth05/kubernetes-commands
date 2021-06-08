# kubernetes-commands

## kubectl

### Get all nodes
```
kubectl get nodes
```

### Get all services
```
kubectl get services
```

### Get all pods
```
kubectl get pod
```

### Status of a pod
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

####  Applying deployment file
```
kubectl apply -f <file_name>
```

#### Deployment configuration file syntax (yaml file)
```
apiVersion: <version>
kind: <what_to_create_Deployment_or_Service>

metadata:
  name: <name_of_deployment>
  labels:
specs:
  replicas: <number_of_replicas_needed>
  selector:
  template:
```

- The file has mainly three parts,
  - metadata
  - specification
  - status (managed by kubernetes, kubernetes will always check desired and actual state, if they are different then kubernetes will try to fix it. this is the basis of self healing feature. the data of the status is from etcd)

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
