# Kubernetes

In this directory we have our own Kubernetes configuration files.

## Persistent volume

First we must create a volume in AWS EBS to store Jenkins data.

https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/ebs-creating-volume.html

Write down the Volume ID of the volume you just created to fill in the Kubernetes manifest. Also configure the volume size, for example

```
...
storage: 50Gi
...
```

Create the resource

`$ kubectl create -f jenkins-pv.yaml`

## Persistent volume claim

Set here the same size we have given to the Persistent Volume

```
...
storage: 50Gi
...
```

and create the resource 

`$ kubectl create -f jenkins-pvc.yaml`

## Storage class

We use ** Storage class ** to provision our application for persistence. When we configure the manifest or the Helm template, we can indicate that it uses persistence of this kind of storage so that whenever we request persistence, a volume will be created automatically in AWS and attached to the corresponding container.

We have called the class _aws_ebs_. If you change the name, remember to update the templates that refer to this class.

## Jenkins values

In the file _jenkins_values.yaml_ is the configuration to deploy Jenkins with a Helm template.
