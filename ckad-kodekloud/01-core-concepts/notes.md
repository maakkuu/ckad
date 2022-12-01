# Core concepts


## Replication Controller and Replica Set
- insures that there are desired number of pods. 
  If not, it will spawn pods to have the desire the number of pods
- Replication Controller
  - old technology
- Replica Set
  - new technology

## Namespaces
- isolation of resources in kube cluster
- can have ffg:
  - policies
  - quotas - resources available for usage
  - networking - dns
- switching namespace
  - `kubectl config set-context $(kubectl config current-context) --namespace=dev`
### ResourceQuota
  - an object that limits the resource

## Imperative Commands
- `kubec`
