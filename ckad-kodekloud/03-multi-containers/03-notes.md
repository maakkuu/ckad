# Multi-Container PODS
## Multiple Containers
- containers share the storage, network(can be access via localhost)
- Design Patterns in Multi-Container Pods
  - sidecar
    - container with other function that does not communicate with other service
  - ambassador
    - poxing the connection to others
  - Adapter
    - transform the data
- Example Usage:
  ```yaml
  # pod metadata
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
    - name: nginx #other container
      image: nginx
  ```
## initContainers
- are container that only runs during the init of the po and terminated once done
- Example Usage:
  ```yaml
  # pod metadata
  spec:
    initContainers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep","1000"] # sleeps 1000 secs before being terminated and nginx container will start
    containers:
    - name: nginx #other container
      image: nginx
  ```