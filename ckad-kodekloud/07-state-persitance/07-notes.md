#  Docker Storage

## Layered Architecture
- 

## Docker Volumes
- persistence storage
- `docker volume create data_volume`
- `docker run -v data_volume:/var/lib/mysql mysql`
  - mounts the `/var/lib/mysql`(path of the container storage) to the `data_volume`
  
### Types of mounting
- volume mounting
  - mounts container dir to docker voluem
- bind mounting
  - mounting container dir to a host dir
### New way of mounting in docker
- `docker run --mount type=bind|volume,source=<dir of the host>,target=<dir of the cont> <image-name>`
### Responsible for storage stuff in docker
- Storage drivers
  - AUFS, ZFS, BTRFS, Device Manager, Overlay, Overlay2
  - is depending on which OS the docker is running

### Storage Drivers
- helps to manage the storage in docker
### Volume Drivers
- handles manage the volumes
- default volumer driver is Local but there are a lot if vol driver 




## Kubernetes Volume & Mounts
### Using of volumes within pod def file
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-name
spec:
  containers:
    - image: nginx
      name: nginx
      volumeMounts: #mounts the /opt path of the container to the volume, data-volume
        - mountPath: /opt 
          name: data-volume
  volumes: # creates volume
    - name: data-volume 
      hostPath: # configure to use a dir in a host. On this case it's in the node
        path: /data #the path
        type: Directory
    - name: data-volume-aws #external storage solution
      awsElasticBlockStore: # aws elastic block volume
        volumeID: <vol-id>
        fsType: ext4

```
### Persistent Volume(pv)
- a cluster wide of pool of volumes
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol
spec:
  accessModes: # How a volume can be mount to the host ReadOnlyMany|ReadWriteOnce|ReadWriteMany
    - ReadWriteOnce
  capacity: 
    storage: 1Gi # capacity to be reserve
  hostPath: 
    path: /tmp/data
  # storage solution can be used instead of hostPath
  awsElasticBlockStore: # aws elastic block volume
    volumeID: <vol-id>
    fsType: ext4
```
### Persistent Volume Claim(pvc)
- bounded to the persistent volume
- automatically bound to a pv based on the ffg
  - Sufficient Capacity
  - Access Modes
  - Volume Modes
  - Storage Class
  - Selector
- 1 to 1 relationship with pv
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-name
spec:
  accessModes: # How a volume can be mount to the host ReadOnlyMany|ReadWriteOnce|ReadWriteMany
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```
- What happens when pv is deleted?
  - configurable via `persistentVolumeReclaimPolicy` valid values are
    - Delete - automatically deleted 
    - Retain - needs to be manually deleted
    - Recycle - data will be scraped and can be use by the other pvc
- pvc in pods
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim: # pvc
        claimName: myclaim  # pvc name
```
## StorageClass
- automatically provision storage
- p