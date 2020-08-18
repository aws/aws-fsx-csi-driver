## Dynamic Provisioning Example
This example shows how to create a FSx for Lustre filesystem using persistence volume claim (PVC) and consumes it from a pod. 


### Edit [StorageClass](./specs/storageclass.yaml)
```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fsx-sc
provisioner: fsx.csi.aws.com
parameters:
  subnetId: subnet-056da83524edbe641
  securityGroupIds: sg-086f61ea73388fb6b
  deploymentType: PERSISTENT_1
  storageType: HDD
```
* subnetId - the subnet ID that the FSx for Lustre filesystem should be created inside.
* securityGroupIds - a common separated list of security group IDs that should be attached to the filesystem
* deploymentType (Optional) - FSx for Lustre supports three deployment types, SCRATCH_1, SCRATCH_2 and PERSISTENT_1. Default: SCRATCH_1.
* kmsKeyId (Optional) - for deployment type PERSISTENT_1, customer can specify a KMS key to use.
* perUnitStorageThroughput (Optional) - for deployment type PERSISTENT_1, customer can specify the storage throughput. Default: "200". Note that customer has to specify as a string here like "200" or "100" etc.
* storageType (Optional) - for deployment type PERSISTENT_1, customer can specify the storage type, either SSD or HDD. Default: "SSD"
* driveCacheType (Required if storageType is "HDD") - for HDD PERSISTENT_1, specify the type of drive cache, either NONE or READ.

### Edit [Persistent Volume Claim Spec](./specs/claim.yaml)
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fsx-claim
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: fsx-sc
  resources:
    requests:
      storage: 6000Gi
```
Update `spec.resource.requests.storage` with the storage capacity to request. The storage capacity value will be rounded up to 1200 GiB, 2400 GiB, or a multiple of 3600 GiB for SSD. If the storageType is specified as HDD, the storage capacity will be rounded up to 6000 GiB or a multiple of 6000 GiB if the perUnitStorageThroughput is 12, or rounded up to 1800 or a multiple of 1800 if the perUnitStorageThroughput is 40.

### Deploy the Application
Create PVC, storageclass and the pod that consumes the PV:
```sh
>> kubectl apply -f examples/kubernetes/dynamic_provisioning/specs/storageclass.yaml
>> kubectl apply -f examples/kubernetes/dynamic_provisioning/specs/claim.yaml
>> kubectl apply -f examples/kubernetes/dynamic_provisioning/specs/pod.yaml
```

### Check the Application uses FSx for Lustre filesystem
After the objects are created, verify that pod is running:

```sh
>> kubectl get pods
```

Also verify that data is written onto FSx for Luster filesystem:

```sh
>> kubectl exec -ti fsx-app -- tail -f /data/out.txt
```
