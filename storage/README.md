## Persistent Storage Setup on K3D

By default K3D comes with persistent localstorage, if you wat use other than localstorage then you can use NFS based Persistent Storage in K3D.

But there is a challange to setup NFS based Persistent Storage setup in K3D. As k3d image does not have nfs-utils, so need to rebuild docker images with Dockerfile and tage with same name and version. Then delete cluster and create again. 
