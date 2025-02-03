For AWS specific deployment
examples > complete > main.tf

aws iam create-service-linked-role --aws-service-name es.amazonaws.com

Expose the openmetadata via external loadbalancer
> kubectl expose svc openmetadata --type=LoadBalancer --name=openmetadata-lb --port=80 --target-port=8585 -n openmetadata
This command will create the Classic loadbalancer internet facing and it works


To expose service via Application Load Balancer 
1) Create the service account - refer service_account_alb.yaml in example/complete/
2) Add the tag "kubernetes.io/role/elb = 1" to the public subnet where ALB to be created
3) Add the aws-load-balancer-controller to the EKS Cluster 
    helm upgrade --install aws-load-balancer-controller eks/aws-load-balancer-controller \
    --set clusterName=open-metadata \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    -n kube-system
4) Create the ingress service in the namespace=openmetadata - refer openmetadata-ingress.yaml example/complete/

This will create the loadbalance in the public subnet.
To access the URL use the ALB DNS name

- path: /openmetadata-deps-web
            pathType: Prefix
            backend:
              service:
                name: openmetadata-deps-web
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: openmetadata-deps-web
                port:
                  number: 8585