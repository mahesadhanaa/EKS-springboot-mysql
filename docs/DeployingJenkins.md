# Deploying Jenkins

We will deploy Jenkins using a Helm Chart. To do this, we previously need to have deployed Helm.

## Persistence

We are going to create a Persistence Volume file so that Jenkins's co-configuration is persistent and not lost.

Follow the steps shown here.

https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/ebs-creating-volume.html

Retrieve the VolumeID you just created and replace it in the following template:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv
  labels:
    type: jenkins-volume
spec:
  capacity:
    storage: TAMAÑO en GB 
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  awsElasticBlockStore:
    volumeID: ID del Volumen de EBS
    fsType: ext4
```

then configure the storage claim

```
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkins-pvc
  labels:
    type: jenkins-volume
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: TAMAÑO en GB
```

Finally apply the changes to the cluster:

`$ kubectl create -f jenkins-pv.yaml -f jenkins-pvc.yaml`

Now the volume of EBS we have created is linked to the EC2 instance where Jenkins is. We have also configured it to retain the information, that is, in case of detre and we have to deploy Jenkins again, it will not erase the information found in the volume.

## Fichero jenkins-values.yaml

This file is where we are going to configure Jenkins for deployment. Is set:

* AdminPassword: Administrator password
* JavaOpts: Options that are passed to the Java virtual machine. For example memory limits or _timezone_
* Plugins:
- cloudbees-folder
    - command-launcher
    - config-file-provider
    - kubernetes-cli
    - kubernetes-credentials
    - mailer
    - ssh-agent
    - amazon-ecr
    - github
    - copyartifact
* Persistence: We add the persistence that we have created in the previous section.
* rbac: we set it to true so Jenkins can handle the Kubernetes API.

## Installing Jenkins

Once the values ​​file is configured, we can deploy Jenkins:

`$ helm install --name jenkins -f jenkins-values.yaml stable/jenkins`

If we are redeploying, we must adjust the value of _Jenkins URL_ in the configuration.

When ready, we can access using the URL of the balancer that has been created.
