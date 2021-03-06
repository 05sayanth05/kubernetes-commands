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

#### Get the deployment config after deployment (from etcd) with status info
```
kubectl get deployment <deployment_name> -o yaml
```

#### Delete deployment using config file
```
kubectl delete -f <deployment_file_name>
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
        env:
        - name: <env_name>
          value: <env_value>
        - name: <env_name>
          valueFrom:
            secretKeyRef:
              name: <name_of_secret>
              key: <key_in_the_secret_file_to_reference>
        - name: <env_name>
          valueFrom:
            configMapKeyRef:
              name: <name_of_configmap>
              key: <key_in_the_configmap_file_to_reference>
```

- The file has mainly three parts,
  - metadata
  - specification
  - status (managed by kubernetes, kubernetes will always check desired and actual state, if they are different then kubernetes will try to fix it. this is the basis of self healing feature. the data of the status is from etcd)
- The things under template is applied for a pod
- Label and selector defines the connection between services and deployments (its pods also)
- The selector under spec matches all the pods with the corresponding label to this deployment

### Service

#### Internal service

##### Syntax of service file for deployment
```
apiVersion: <version>
kind: Service

metadata:
  name: <internal_service_name>

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

#### External service

##### Syntax of service file for deployment
```
apiVersion: <version>
kind: Service

metadata:
  name: <external_service_name>

spec:
  selector:
    <key>: <value>
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: <port_in_which_the_service_should_listen_for_requests>
      targetPort: <port_in_which_the_pod_is_listening_that_is_containerPort>
      nodePort: <port_for_the_external_ip>
```

- `nodePort` will only accept values from `30000` to `32767`
- `type` is set to `LoadBalancer` for creating external service

#### Config file syntax for service with multiple ports opened
```
apiVersion: <version>
kind: Service

metadata:
  name: <service_name>

spec:
  selector:
    <key>: <value>
  ports:
    - name: <port_name>
      protocol: <protocol_ex:TCP>
      port: <port_number>
      targetPort: <port_in_which_the_pod_is_listening_that_is_containerPort>
    - name: <port_name>
      protocol: <protocol_ex:TCP>
      port: <port_number>
      targetPort: <port_in_which_the_pod_is_listening_that_is_containerPort>
```

- This is same for both internal and external service

##### Set external ip for an external service in minikube
```
minikube service <service_name>
```

#### Get services
```
kubectl get service
```

#### Describe a service
```
kubectl describe service <service_name>
```

#### Delete service using config file
```
kubectl delete -f <service_file_name>
```

### Ingress

- External service are mainly used in development environment in order to test the application quickly
- In production environment ingress should be used

#### Syntax
```
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: <ingress_name>

spec:
  tls:
  - hosts:
    - <host_name:ex:host.com>
    secretName: <secret_name>
  rules:
  - host: <host_name_ex:host.com>
  - http:
      paths:
      - path: <path_ex:/>
        pathType: <type_of_path_ex:Prefix>
        backend:
          service:
            name: <service_name>
            port:
              number: <port_of_service>
        
```

- The `http` attribute does not correspond to the incoming user request
- The value of `host` attribute should be a valid domain address
  - This domain name should be mapped to the ip address of the node which is the entry point to the k8s cluster
  - The entrypoint to the cluster will be the node with ingress controller pods

#### Install ingress in minikube
```
minikube addons enable ingress
```

- This will enable the k8s nginx implementation of ingress

#### TLS secret syntax
```
apiVersion: v1
kind: Secret

metadata:
  name: <secret_name>
  namespace: <namespace_in_which_the_corresponding_ingress_is_defined>

data:
  tls.crt: <base64_encoded_certificate>
  tls.key: <base64_encoded_key>

type: kubernetes.io/tls
```

### Secret

#### Syntax of secret file
```
apiVersion: v1
kind: Secret

metadata:
  name: <name_of_secret>
type: <type_of_secret>

data:
  <key>: <base64_value>
  <key>: <base64_value>
```

- The `type` will generally have `Opaque` as value, this specifies secret is key value store 
- The secrets have to be created before deployment in the cluster

#### Creating secret in the cluster using config file
```
kubectl apply -f <secret_file_name>
```

#### Get secrets
```
kubectl get secret
```

### ConfigMap

#### Syntax
```
apiVersion: v1
kind: ConfigMap

metadata:
  name: <configmap_name>
  namespace: <namespace_name>

data:
  <key>: <service_name>.<value>
```

- The ConfigMap is a centralized data store for reference
- This should be created before deployment because we are referring data for this

#### Creating config map from the config file
```
kubectl apply -f <configmap_file_name>
```

#### Get components from a namespace
```
kubectl get <component_name_ex:pod> -n <namespace_name>
```

### Namespace

- It is used to organize resources
- It is a virtual cluster inside a kubernetes cluster
- By default there are four namespaces

#### Get namespaces
```
kubectl get namespace
```

#### Create namespace using kubectl
```
kubectl create namespace <name_of_namespace>
```

- Namespace can be created through config files

#### Get info about components in a namespace
```
kubectl get <component_ex:pod> -n <namespace_name>
```

#### Syntax of yaml file
```
apiVersion: v1
kind: Namespace
metadata:
  name: <namespace_name_here>
```

### Data persistance

- Kubernetes only provide the interface to storage, the selection and management of storage have to be done manually

#### Persistance volume

- PVs are not namespaced. They belong to the whole cluster

##### Syntax
```
apiVersion: v1
kind: PersistentVolume

metadata:
  name: <pv_name>

spec:
  capacity:
    storage: <storage_ex:5Gi>
  volumeMode: <mode>
  accessMode:
    - <accessMode1>
  
```

- **not compleated**

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
