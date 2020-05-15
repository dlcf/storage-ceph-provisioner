# storage-ceph-provisioner

## Create a Ceph admin secret
```shell
ceph auth get-key client.admin > /tmp/secret
kubectl create ns cephfs
kubectl create secret generic ceph-secret-admin --from-file=/tmp/secret --namespace=cephfs
```

## Install with RBAC roles
```shell
cd ceph/cephfs/deploy
NAMESPACE=cephfs # change this if you want to deploy it in another namespace
sed -r -i "s/namespace: [^ ]+/namespace: $NAMESPACE/g" ./rbac/*.yaml
sed -r -i "N;s/(name: PROVISIONER_SECRET_NAMESPACE.*\n[[:space:]]*)value:.*/\1value: $NAMESPACE/" ./rbac/deployment.yaml
kubectl -n $NAMESPACE apply -f ./rbac
```
## Replace Ceph monitor's IP in /tmp/class.yaml 
```shell
cat > /tmp/class.yaml << \EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: cephfs
provisioner: ceph.com/cephfs
parameters:
    monitors: 127.0.0.1:6789
    adminId: admin
    adminSecretName: ceph-secret-admin
    adminSecretNamespace: "kube-system"
    claimRoot: /pvc-volumes

```
