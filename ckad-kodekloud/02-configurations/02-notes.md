# 02 Configuration

## CMD and ARGs in docker
- specifying cmd on docker
  -`docker run ubuntu <CMD`
     -  e.g: `docker run ubunt sleep 5`
  - via dockerfile
  ```Dockefile
  FROM Ubuntu
  
  CMD <cmd> <param1>
  // or
  CMD ["<cmd>","<param>"]
  
  ```
- `ENTRYPOINT`
  - a cmd that gets execute during the docker run
  - allows a param to be concatenated during docker run
  ```
  FROM ubuntu
  
  ENTRYPOINT ["sleep"]

  ```
  - can be run using `docker run ubuntu-sleeper 10`
  this is equals to `docker run ubuntu-sleeper sleep 10`
  - to have the default value you can use the `CMD`
  - e.g
  ```Dockerfile
  FROM ubuntu
  
  ENTRYPOINT ["sleep"]
  
  CMD ["5"]
  ```
  - if `docker run ubuntu-sleeper` is used the default param of `sleep` cmd is `5` 
  since it was indicated in `CMD` in dockerfile
## CMD and ARGS in kube
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: ubuntu-sleeper-tst
  name: ubuntu-sleeper-tst
spec:
  containers:
  - image: ubuntu-sleeper
    name: ubuntu-sleeper-tst
    command: ["sleep2.0"] \\overides the Dockerfile's ENTRYPOINT
    args: ["10"] \\overides the Dockerfile's CMD
```
---
## Environment Variables
- setting env var value in pod def file
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
    run: ubuntu-sleeper-tst
    name: ubuntu-sleeper-tst
  spec:
    containers:
     - image: ubuntu-sleeper
       name: ubuntu-sleeper-tst
       env:
         - name: ENV_NAME
           value: ENV_VALUE

  ```
---
## ConfigMap
- object that holds key-value pairs
  - steps on using configmap
    1. create configmap object
       - creating configmap `kubectl create configmap
         <configmap-name> --from-literal=ENV_VAR1=ENV_VAL1 
       --from-literal=ENV_VAR2=ENV_VAL2`
       - via def file
       ```yaml
       apiVersion: v1
       kind: ConfigMap
       metadata:
         name: config-map-name
       data:
         ENV_NAME1: ENV_VAL1
         ENV_NAME2: ENV_VAL2

       ```
    2. map/connect/inject the configmap to the pod
       ```yaml
        #... pod definition 
        spec:
          containers:
          - image: ubuntu-sleeper
            name: ubuntu-sleeper-tst
            envFrom:
              - configMapRef:
                  name: <config-map-name> 
       ```
---
## Secret
- same as config map but for confidential data
- stored data should be base64 format
- def file
       ```yaml
       apiVersion: v1
       kind: Secret
       metadata:
         name: config-map-name
       data:
         ENV_NAME1: BASE64_ENV_VAL1
         ENV_NAME2: BASE64_ENV_VAL2
      ```
- map/connect/inject the configmap to the pod
  ```yaml
  #... pod definition
  spec:
  containers:
  - image: ubuntu-sleeper
  name: ubuntu-sleeper-tst
  envFrom:
  - secretRef:
      name: <secret-name> 
  ```
- cmd
  - `kubectl create secret <type> <secret-name>`
---
## Security in Docker
- container and the host share the same kernel
- by default container use the `root` user
  - `root` is the most powerful user in an OS
  it have all the capabilities 
- use different user other that the `root`
  - `docker container run --user=<user-id> ubuntu` 
  - in Dockerfile
  ```Dockerfile
  FROM ubuntu
  
  USER <user-id>
  ```
  
## Security Context
- security context can be set via pod and container lvl
- setting the security context in definition file
  ```yaml
  #   ..pod metadata
  spec:
    securityContext:
      runAsUser: 1000 #1000 is a userId
    containers:
      - name: nginx
        image: nginx
        securityContext:
          runAsUser: 1000 #1000 is an userId
          capabilities:
            add: ["MAC_ADMIN"] #adding capabilities
  ```
  ---
## Resource Requirements
- `scheduler` is responsible on which node will the pod be spawned
- resources
  - cpu 
    - by core/hyperthread
    - 1 = 1 core
    - .1 = 100m
  - mem
    - by Mi,Gi,Ki 
  - disk
- resource request
  - resource being request to allow the pod to run
  - cpu
    - 0.5 default
  - mem
    - 256 Mi default
  - disk
- setting resources config
  ```yaml
  # pod metadata
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
      resources:
        requests: # resource required for pod to up
          memory: "1Gi"
          cpu: 1
        limits: # resource limit of running pod, if reached the pod will be terminated
          memory: "2Gi"
          cpu: "2"
    
  ```
---
## Service Account
- an account used by an app/service to access kube api
- `kubectl create serviceaccount sa-name`
  - under the hood it will also create a secret 
  - the secret can be use as Bearer token .e.g "Authorization: Bearer <token"
- mounting a serviceAccount
  ```yaml
  # pod metadata
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
    serviceAccountName: <service-account-name>
    autoMountServiceAccountToken: false
  ```
- v1.24 update
  - svc-account not
  - svc-account no token is automatically created
    so you need to create a token separately
    - `kubectl create serviceaccount <svc-acct-name>`
    - `kubectl create token <svc-acct-name>`
    - 
- service account token secret is not recommended anymore
## Node Affinity 
- Handle the advance way of assigning a pod placement in a node
- Example Usage:
  ```yaml
  # pod metadata
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
    affinity:
     nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: size #label of the node
              operator: Exits # In/NotIn but requires another param `value` 
              #value: this is needed if the operator needs a values to compared with
  ```
- what if node affinity
  - `requiredDuringSchedulingIgnoredDuringExecution`
    - if it cannot find a node with right affinity it pod will not be schedules
  - `prefferedDuringSchedulingIgnoredDuringExecution`
    - if it cannot find a node with right affinity it will place the pod in any node
## Taints & Toleration vs Node Affinity 
- Taints & Toleration
  - pod can be placed to node without taints
- Node Affinity
  - pod can be place to other node


