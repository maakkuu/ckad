# Pod Design

### Labels and selectors
- Labels
  - use to group resources
  - under `metadata.labels`
- Selectors
  - use to filter resource
  - example usage: `kubectl get po --selector app=auth-service`
    - filters all pod with label of `app=auth-service`
    
- Annotations
  - use to record other information about the resources

### Update and Rollout
- rollout
  - is when a new deployment is deployed
- `kubectl rollout status deployment <deployment-name>`
  - to see the rollout status
- `kubectl rollout history deployment <deployment-name>`
  - to see the rollout history

- 2 types of deployment strategy
  - Recreate Strategy
    - remova all older version then deploy new version
    - Example
    ```yaml
    # deployment metadata
    spec:
      strategy:
        type: Recreate
    ```
  - Rolling Update
    - Take down one older version and deploy the new version continuously 
    until all older version has been changed by the new version
    - Example
    ```yaml
    # deployment metadata
    spec:
      strategy:
        type: RollingUpdate
    ```
- `kubect apply -f <deployment-def.yaml> --record`
  - automatically create a rollout
  - add `--record` flag to add a `change-cause` to the rollout history
    - `change-cause` will show the command that applied the rollout
- `kubectl set image deploymrny/myapp-deployment nginx=nginx:1.9.1 --record`
  - automatically create a rollout
  - changing the image of the deployment from `nginx`=`nginx:1.9.1`
  - `change-cause` will show the command that applied the rollout
- Deployment under the hood
  - New Replica set is created to deploy the new version
- Rollback
  - `kubect rollout undo deployment/myapp-deployment`
    - this rollback the deployment to the previous version
  
### Jobs
- `spec.restartPolicy` defines if the pod will restart upon exit, valid values are `Always`|`Never`
- job definition file
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: <job-name>
spec:
  completions: 3 # number of desired completed jobs
  parallelism: 3 # number of jobs to run in parallel 
  template:
    spec:
      containers:
        - name: nginx
          image: nginx
          command: ['sleep', '10']
      restartPolicy: Never
```
- `kubect create -f <job-def-file>`

### CronJobs
- job definition file
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: <job-name>
spec:
  schedule:
  jobTemplate:
    spec: 
      completions: 3 # number of desired completed jobs
      parallelism: 3 # number of jobs to run in parallel 
      template:
        spec:
          containers:
            - name: nginx
              image: nginx
              command: ['sleep', '10']
          restartPolicy: Never
```
- `kubectl get cronjobs`
  
  