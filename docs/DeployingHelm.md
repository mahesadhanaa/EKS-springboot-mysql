# Helm - Package Manager

## Install Helm

- To download:

https://github.com/kubernetes/helm/releases

- Unzip

`$ tar -zxvf helm-$VERSION-linux-amd64.tgz`

- Put the binary in its place

`$ sudo mv linux-amd64/helm /usr/local/bin`

## Authorization

Run this line to grant authorization to Helm

`$ kubectl create -f https://raw.githubusercontent.com/nordri/kubernetes-experiments/master/Pipeline/ServiceAccount.yaml`

## Start 

`$ helm init --service-account tiller `

- Check

`$ kubectl --namespace=kube-system get pods --selector app=helm`

```
NAME                            READY     STATUS    RESTARTS   AGE
tiller-deploy-78f96d6f9-8kc88   1/1       Running   0          2m
```

