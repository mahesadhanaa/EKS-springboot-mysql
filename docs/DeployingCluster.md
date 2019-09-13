# Cluster deployment

We will deploy the Kubernetes cluster using Amazon Elastic Container Service for Kubernetes (EKS). For this we need an AWS account.

We will use Amazon CLI to provision the Cluster, to install it follow the documentation:

https://docs.aws.amazon.com/es_es/cli/latest/userguide/installing.html

## Input data

Cluster Name: Fortypersona

## Role creation

We need an AWS role with the ability to create resources.

```
cat >policy.json<<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:UpdateAutoScalingGroup",
                "ec2:AttachVolume",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:CreateRoute",
                "ec2:CreateSecurityGroup",
                "ec2:CreateTags",
                "ec2:CreateVolume",
                "ec2:DeleteRoute",
                "ec2:DeleteSecurityGroup",
                "ec2:DeleteVolume",
                "ec2:DescribeInstances",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSubnets",
                "ec2:DescribeVolumes",
                "ec2:DescribeVolumesModifications",
                "ec2:DescribeVpcs",
                "ec2:DetachVolume",
                "ec2:ModifyInstanceAttribute",
                "ec2:ModifyVolume",
                "ec2:RevokeSecurityGroupIngress",
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
                "elasticloadbalancing:AttachLoadBalancerToSubnets",
                "elasticloadbalancing:ConfigureHealthCheck",
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:CreateLoadBalancer",
                "elasticloadbalancing:CreateLoadBalancerListeners",
                "elasticloadbalancing:CreateLoadBalancerPolicy",
                "elasticloadbalancing:CreateTargetGroup",
                "elasticloadbalancing:DeleteListener",
                "elasticloadbalancing:DeleteLoadBalancer",
                "elasticloadbalancing:DeleteLoadBalancerListeners",
                "elasticloadbalancing:DeleteTargetGroup",
                "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
                "elasticloadbalancing:DeregisterTargets",
                "elasticloadbalancing:DescribeListeners",
                "elasticloadbalancing:DescribeLoadBalancerAttributes",
                "elasticloadbalancing:DescribeLoadBalancerPolicies",
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticloadbalancing:DescribeTargetGroupAttributes",
                "elasticloadbalancing:DescribeTargetGroups",
                "elasticloadbalancing:DescribeTargetHealth",
                "elasticloadbalancing:DetachLoadBalancerFromSubnets",
                "elasticloadbalancing:ModifyListener",
                "elasticloadbalancing:ModifyLoadBalancerAttributes",
                "elasticloadbalancing:ModifyTargetGroup",
                "elasticloadbalancing:ModifyTargetGroupAttributes",
                "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
                "elasticloadbalancing:SetLoadBalancerPoliciesOfListener",
                "kms:DescribeKey"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "*",
            "Condition": {
                "StringLike": {
                    "iam:AWSServiceName": "elasticloadbalancing.amazonaws.com"
                }
            }
        }
    ]
}
EOF
```

`$ aws iam create-role --role-name KubernetesRole --assume-role-policy-document file:///PATH/TO/policy.json`

## Create the VPC

We will configure a Virtual Private Cloud for Kubernetes. This will not give more insulation in the containers.

```
$ cat >parameters.json<<EOF
[
  {"ParameterKey":"VpcBlock","ParameterValue":"192.168.0.0/16"},
  {"ParameterKey":"Subnet01Block","ParameterValue":"192.168.64.0/18"},
  {"ParameterKey":"Subnet02Block","ParameterValue":"192.168.128.0/18}"},
  {"ParameterKey":"Subnet03Block","ParameterValue":"192.168.192.0/18"}
]
EOF
```

```
$ aws cloudformation create-stack \
  --stack-name Fortypersona \
  --template-url https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2018-11-07/amazon-eks-vpc-sample.yaml \
  --parameters file:///PATH/TO/parameters.json \
  --disable-rollback
```

## Create the Cluster Master

Now that we have the network and the role configured we can create the master of the cluster, it is the machine that will control the cluster, replace the values ​​with the appropriate ones in each case.

```
$ aws eks create-cluster \
  --name Fortypersona \
  --role-arn arn:aws:iam::303679435142:role/KubernetesRole \
  --resources-vpc-config subnetIds=subnet-08415098923def71a,subnet-0d5fc19ee2f96d6f1,subnet-0e5a5155f2406e49d
```

## Connect our _kubectl_ to the cluster

In order to authenticate against the cluster we have to follow these steps:

https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html#eks-configure-kubectl

Once we have resolved the dependencies we can execute:

`$ aws eks update-kubeconfig --name Fortypersona`

## Create cluster nodes

Cluster nodes are created using a CloudFormation template that configures a scalability group. That is to say, it facilitates the creation of a dynamic cluster that can scale although for the moment it only scales manually. It starts with a single mode, if more nodes are desired, set the `NodeAutoScalingGroupMaxSize` parameter.

We create the file with the parameters:

```
$ cat >parameters.json <<EOF
[
  {"ParameterKey":"BootstrapArguments","ParameterValue":""},
  {"ParameterKey":"ClusterControlPlaneSecurityGroup","ParameterValue":"ID Del grupo de seguridad por defecto de la VPC"},
  {"ParameterKey":"ClusterName","ParameterValue":"Fortypersona"},
  {"ParameterKey":"KeyName","ParameterValue":"RSA Key para acceder a los nodos"},
  {"ParameterKey":"NodeAutoScalingGroupMaxSize","ParameterValue":"1"},
  {"ParameterKey":"NodeAutoScalingGroupMinSize","ParameterValue":"1"},
  {"ParameterKey":"NodeGroupName","ParameterValue":"FortyWorkers"},
  {"ParameterKey":"NodeImageId","ParameterValue":"ami-00c3b2d35bddd4f5c"},
  {"ParameterKey":"NodeInstanceType","ParameterValue":"c5.2xlarge"},
  {"ParameterKey":"NodeVolumeSize","ParameterValue":"20"},
  {"ParameterKey":"Subnets","ParameterValue":"subnet-08415098923def71a"},
  {"ParameterKey":"VpcId","ParameterValue":"vpc-0f059d53d0bfdc9ed"}
]
EOF
```

And we create the scalability group

```
$ aws cloudformation create-stack \
  --stack-name ${CLUSTER_NAME} \
  --template-url https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2018-11-07/amazon-eks-nodegroup.yaml \
  --parameters file:///PATH/TO/parameters.json \
  --disable-rollback \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```

To allow the nodes to join the Kubernetes cluster:

We take the RNA varlor that has generated the CloudFormation:

`$ ARN_ROLE=$(aws cloudformation describe-stacks --stack-name Fortypersona | jq --raw-output ' .Stacks[] | .Outputs[] | select(.OutputKey | contains("NodeInstanceRole")) | .OutputValue' )`

and we generate a Config Map:

```
$ cat >aws-auth-cm.yaml<<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: ${ARN_ROLE}
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
EOF
```

`$ kubectl apply -f aws-auth-cm.yaml`
