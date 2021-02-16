# SkyhookDM-Arrow Docker 

Docker image containing SkyhookDM built on top of Arrow along with C++ and Python API clients.

### Deploying SkyhookDM-Arrow on a Rook cluster
* Change the Ceph image tag in the Rook CRD [here](https://github.com/rook/rook/blob/master/cluster/examples/kubernetes/ceph/cluster.yaml#L24) to the image built from [this](./docker) dir (or you can quickly use `uccross/skyhookdm-arrow:vX.Y.Z` as the image tag) to change your Rook Ceph cluster to the `vX.Y.Z` version of SkyhookDM Arrow. 

* After the cluster is updated, we need to deploy a Pod with the PyArrow (with RadosParquetFileFormat API) library installed to start interacting with the cluster. This can be achieved by following these steps:

  1) Update the ConfigMap with configuration options to be able to load the arrow CLS plugins.
  ```bash
  kubectl apply -f cls.yaml
  ```

  2) Create a Pod with PyArrow pre-installed for connecting to the cluster and running queries. 
  ```bash
  kubectl apply -f client.yaml
  ```

  3) Create a CephFS on the Rook cluster.
  ```bash
  kubectl create -f filesystem.yaml
  ```
  
  4) Copy the Ceph configuration and Keyring from some OSD/MON Pod to the playground Pod.
  ```bash
  # copy the ceph config
  kubectl -n rook-ceph cp rook-ceph-osd-X-YYYYYYYY-ZZZZZ:/var/lib/rook/rook-ceph/rook-ceph.config ceph.conf
  kubectl -n rook-ceph cp ceph.conf rook-ceph-playground:/etc/ceph/ceph.conf

  # copy the keyring
  kubectl -n rook-ceph cp rook-ceph-osd-X-YYYYYYYY-ZZZZZ:/var/lib/rook/rook-ceph/client.admin.keyring keyring
  kubectl -n rook-ceph cp keyring rook-ceph-playground:/etc/ceph/keyring
  ```

  5) Check the connection to the cluster from the client Pod.
  ```bash
  # get a shell into the client pod
  kubectl -n rook-ceph exec -it rook-ceph-playground bash

  # check the connection status
  $ ceph -s
  ```

  6) Now, install `ceph-fuse` and mount CephFS into some path in the client Pod using it. [In a later release 
  `ceph-fuse` will come installed in the SkyhookDM image itself.]

  ```bash
  yum install ceph-fuse
  mkdir -p /mnt/cephfs
  ceph-fuse --client_fs cephfs /mnt/cephfs 
  # the client_fs value can be different. Please check the filesystem.yaml file for details.
  ```

  4) Download some example dataset into `/path/to/cephfs/mount`. For example,
  ```bash
  cd /mnt/cephfs
  wget https://raw.githubusercontent.com/JayjeetAtGithub/zips/main/nyc.zip
  unzip nyc.zip
  ```

  4) Modify the [example python script](./example.py) according to your needs and execute.
  ```bash
  python3 example.py
  ```
