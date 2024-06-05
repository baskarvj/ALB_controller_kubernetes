### Create policy for ALB controller service account
ref: https://docs.aws.amazon.com/eks/latest/userguide/lbc-helm.html

create policy --> paste the code from below url and  --> give name --> AWSLoadBalancerControllerIAMPolicy --> create

https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.7.2/docs/install/iam_policy.json

create role --> web_identity --> select your cluster oidc --> select policy --> AWSLoadBalancerControllerIAMPolicy --> give name for role --> AWSLoadBalancerControllerIAMrole

### create value file for ALB controller service account

~~~
vi alb-sa.yml
~~~
~~~
serviceAccount:
  create: true
  name: aws-load-balancer-controller
  nameTest:
  annotations:
    eks.amazonaws.com/role-arn: <enter_your_alb_controller_role_arn_here>

clusterName: <enter_your_cluster_name>
~~~

### Add ALB controller chart to helm repo
~~~
helm repo add eks https://aws.github.io/eks-charts
~~~
### update helm repo
~~~
helm repo update eks
~~~
### install alb controller
~~~
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --values=alb-sa.yml
~~~

### verify using 
~~~
kubectl get sa -n kube-system |grep aws-load-balancer-controller
~~~
~~~
kubectl get deployment -n kube-system aws-load-balancer-controller
~~~

