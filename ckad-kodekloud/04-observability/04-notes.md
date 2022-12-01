# Observability
## Pod Lifecycles
- Pending
  - scheduler finding to which node is to be placed
- ContainerCreating
  - pulling the image
- Running
  - running all containers
## Pod Conditions
- PodScheduled
- Initialized
- ContainerReady
- Ready
## Readiness Probes
- indicates that the pod is running and can accept now a request traffic
- Example Usage:
  ```yaml
  # pod metadata
  spec:
   containers:
    - name: nginx #other container
      image: nginx
      readinessProbe:
        httpGet: 
          path: /api/ready
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 8
  ```
## Liveness Probes
- test the container if it's healthy if it's not it pod will be recreated
- Example Usage:
  ```yaml
  # pod metadata
  spec:
   containers:
    - name: nginx #other container
      image: nginx
      livenessProbe:
        httpGet: 
          path: /api/ready
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 8
  ```
## Logging
- `kubectl logs -f <pod-name>`
- `kubectl logs -f <pod-name> -c <container>`
- `kubectl logs -f <pod-name> <container>`

## Monitoring
- metric-server an in-memory storage
  - How does it work?
    - `kubelet` 
      - a running agent on every node who's responsible for receiving cmd from master node
      - kubelet has sub-compenent `c-advisor` exposed node metric to metric-server
- Usage
  - clone the definition files of the metric server
    - `git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git`
  - apply all def files
  - check metrics
    - `kubectl top <resources>`