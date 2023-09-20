# Docker Plugin
There can be multiple docker plugins. i.e. network,volume, log, authorization, metric collection etc.

# Rootless mode
In this mode both docker daemon and containers both run as non-root user to avoid security related issues.

# Docker Storage
There are 2 types of storages in docker container..i.e. file system storage and volume storage. There are different purpose for these storages.

These storages are handled with inbuilt(local) or external drivers. 

## Storage Drivers
Docker uses layered architecture to build docker imaages. It is to ensure efficient storage. If evey image is stored seprately then host file system will grow duplicating the same files in every image. For example, a user app using operating system image as base image. In this case, there are multiple user apps using same base image. if they are stored as separate files without using layered architecture then disk will run out of space soon.

### Layered Architecture

#### Image layers (ready only)
When a docker image is built, it will create layern for each instruction in **Dockerfile**. For example in below file
````
FROM ubuntu
RUN apt-get update && apt-get install python
RUN pip install flask
COPY . /app/src
ENTRYPOINT FLASK_APP=/app/src/app.py flask run
````
it will create a layer for each instruction in above file. So all 5 instructions will create a layer. 

Each layer is built and cached for subsequent calls. So if you rerun your docker build command for above it will use the cached layers. Thats how it will make efficient use of disk space.

Also note that even if you delete a file in any layer then still it will be persisted in the previous layer.

There are multiple storage driver options available to work with host. For linux, some of them are **overlay, aufs, btrfs, zfs, devicemapper and overlay2** etc.

#### Container layer (writable)
Suppose some file is available in image layer but during container runtime, a modification is required to the file. In this case, a new layer is created called container layer, which is writable.

layered architecture uses copy on write and creates this container layer. it will copy the file to this layer and will make changes. This layer will be removed once the container is removed.

#### Overlay2
Currently **aufs** (works on ubuntu), **devicemapper** works on (RHEL, fedora) are **deprecated** and **not recommended** to use. docker will inspect the operating and choose best recommended storage driver. o**verlay2 is now supported on almost every host** linux operating system and is default **recommended** storage driver. it provides **efficient memory** and better performance trade off. This will use less memory as compared to btrfs and zfs. But **btrfs/zfs** are good for other usecases where file size is large because it uses **page caching**.

overlay2 works on **OverlayFS** linux kernel driver and **xfs** filesystem. It is a union filesystem. It is very efficient for most usecases unless writing to a big file. It uses **copy on write**. it means a file is copied from lowerdirs into upperdir and then modified and kept in the upperdir which obscure the lowerdirs for the changes. so performance hit will be **once when file is copied**. it can be noticable when a **file is large in size otherwise** it very good.

The **image** layer is the **lowerdir** and the **container** layer is the **upperdir**. If the image has multiple layers, multiple **lowerdir** directories are used. The unified view is exposed through a directory called **merged** which is effectively the containers mount point

overlay2 uses multiple directories to manage these layers. it natively supports up to **128 lower** **OverlayFS** layers. there are multiple directories like **lowerdir, upperdir and merged**. User is served the files from **merged**(container mount). it will combine the files from **upperdir**(container layer) and **lowerdirs**(image layer). when file is deleted, a whiteout file is created in upperdir and obscured in lowerdirs.

##### Configuration
it can be configured to use with updating below in `/etc/docker/daemon.json'
````
{
  "storage-driver": "overlay2"
}
````

## Volume Drivers
When containers need to share some data with other containers or use host file system. for example, containers use /etc/resolve.conf to resolve the dns server. it can be mounted to volume. Sometimes container might use the cloud provider's storage to mount into the container.

There are both built in and external/third party plugins.

## Volume Mounts
Also there are 2 types of mounts too for volumes.

### bind mount
When the host directory is mounted into the container using volume then it is called bind mount. Any change done in the container in the mounted path will be reflected in the host filesystem too. Thats why its called bind mount. This bind mount is not managed by docker and user have to manage it. if user doesn't need the volume anymore, it has to be manually deleted. It can be called host volume too.
For example,
````
docker run -v /host-vol:/data/app ubuntu
````
A host dir if not exists will be created with path /host-vol. Please note that host directory is required to be created before this if mount is used. see below
````
docker run --mount type=bind,src=/host-vol,target=/data/app
````
if `/host-vol` doesnt exist, above will throw error 

To get the details about bind volumes, use below command
````
docker inspect my_container --format '{{json .Mounts}}'

````
for bind mounts, use the container id

### volume mount
This is default mount type when not specified. 
if you see below
````
docker run --mount src=local-vol,target=/data/app ubuntu
or
docker run -v local-vol:/data/app ubuntu
````
In this case, type=volume, which is default if not defined. 
From above command, it will create the **managed volume** called **local-vol** under `/var/lib/docker/volumes`. This is managed by docker and can be deleted through `docker volume prune` command. So in this case, user doesn't need to worry where this is created so to delete it manually.
It is called **local volume** and user didnt specify explicit volume and as it doesnt exist already, it will create it.
Another way to use a **named volume**, in this case a volume is created first and then used inside the container.
````
docker volume create named-vol
docker run -v named-vol:/data/app
````
in this case, named-vol is created under `/var/lib/dcoker/volumes' and can also be shared with other containers as it is defined outside of the container.

Please note that mount option is verbose and recommended way to create volumes.

In the above cases, driver used is `local`, which is built in. There are other external drivers too.i.e. nfs,azure,gce,aws,cifs,glusterfs etc.

To inspect the driver and volume related info, run below command on named volumes:
````
docker inspect named-vol
````
it will show the driver and mount details


Another exmaple for aws ebs volume can be as below:
````
docker run --volume-driver rexray/ebs --mount src=ebs-vol,target=/var/lib/mysql
````

### tmpfs mount
There is another type of mount, which is useful for usecases, where is security is tight and no data should be stored in filesystem. In this case, data is stored in memory and deleted when container exits. 
````
docker run --mount type=tmpfs,target=/data/app
````
source is not required as data is not stored on host filesystem.

### npipe
This is applicable for windows containers and uses named pipe for inter container communication.
````
docker run --rm --mount type=npipe,source=namedpipe,target=/container/path microsoft/windowsservercore
````

# Container Storage Interface (CSI)
Kubernetes support multiple runtimes.i.e. docker, crio, rkt etc. This is possible due to CRI (Container runtime interface)

So any runtime engine which confirms to CRI can be used in kubernetes. 

Similarily, any storage driver can be used on kubernetes if it conforms to CSI. For example, RPC calls like below are implemented by storage drivers
- CreateVolume
- Deletevolume
- ControllerPublishVolume


# Volumes
Volumes in kubernetes is like a host/bind mount in docker. where explicit directory is provided. In a pod, as below
````
volumes:
  - name: my-vol
    emptyDir: {}
````
It is an abstraction that represents a storage in kubernetes for a pod. Data is stored till pod is running and shared within the containers in the pod.

## Persistent Volumes (PV)
It is like named volume in docker. It can created independently from pods configuration and managed by kubernetes.
This is kubernetes volumes created for a volume type. For example hostPath. It can be created by administrator to avail the storage for pods.

## Persistent Volume Claim (PVC)
This is the created by user to attach to PV created by administrator. There is 1:1 relationship between PV and PVC. If there is one PV available and 2 PVC are created the 1 PVC will be bounded and another will wait for a PV to be available.

dont confuse with 1:1 relationship between pv and pvc. pods and pvc can have 1:many relationship. which means multiple pods can read from same pv.

### PersistentVolumeClaimPolicy 
pvc policy can be set to Retain, Delete or Recycle

#### Retain
When pvc is deleted, pv is retained and data is not deleted.

# Delete
when pvc is deleted, pv is deleted too.

# Recycle
When pvc is deleted, pv data is scrambled and made available to be reused.

## Storage Class

