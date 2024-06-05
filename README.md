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
### create ingress (for load balancer)
~~~
vi lb.yml
~~~
~~~
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: default
  name: my-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/group.name: my-alb-group  #Use this to share ALB among multiple ingresses. #CostEffective
    alb.ingress.kubernetes.io/load-balancer-name: my-alb  # give ALB a meaningfull name otherwise a random name is assigned by AWS.
    alb.ingress.kubernetes.io/certificate-arn: <cert_arn>
    alb.ingress.kubernetes.io/subnets: subnet-0e1b5b8740e23d44a5, subnet-0374e693583e43421d3, subnet-09912e9f115a463377, subnet-03b9bv704be4ab48abb, subnet-0f34f48e7e003a041a, subnet-0142077dc45ef94325
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'

spec:
  ingressClassName: alb
  tls:
  - hosts:
    - eks.baskeytech.cloud
  defaultBackend:
    service:
      name: my-service
      port:
        number: 80
  rules:
    - host: eks.baskeytech.cloud
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: my-service
              port:
                number: 80
~~~

~~~
kubectl apply -f lb.yml
~~~
~~~
kubectl get ing
~~~
### If got error in ingress controller add this permission in AWSLoadBalancerControllerIAMPolicy policy
~~~
elasticloadbalancing:AddTags
~~~
![image](https://github.com/baskarvj/ALB_controller_kubernetes/assets/103120325/1d18452f-2291-4d77-ab22-31cd0c8f9716)


