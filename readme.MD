# Ingress Controller

REGION_CODE=us-east-1
CLUSTER_NAME=expense
ACC_ID=090808669085

### Setting Permissions

* Create an IAM OIDC provider
```
eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve
```

* Download an IAM policy for the LBC - Load Balancer Controller
```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.13.2/docs/install/iam_policy.json
```

* Create an IAM policy named AWSLoadBalancerControllerIAMPolicy
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

* Create an IAM role and Kubernetes ServiceAccount for the LBC. Provide access to the eks using IAM Policy we created
```
eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::$ACC_ID:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region $REGION_CODE \
--approve
```

### Install Drivers

* Add the EKS chart repo to Helm
```
helm repo add eks https://aws.github.io/eks-charts
```

* Helm install command for clusters with IRSA:
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```