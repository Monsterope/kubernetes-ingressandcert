# Example Kubernete Deployment Service, Nginx Ingress and Cert manager Letsencrypt

File Example Kubernete and git action workflows

## STEP (For AWS EKS)
- *** Recheck setting configure AWS CLI
- *** Recheck install kubectl
- *** Recheck install Helm
- *** Recheck Create AWS EKS
- Load Resource AWS EKS
  - aws eks update-kubeconfig --name {EKS_CLUSTER_NAME} --region {REGION}
- Create namespace and set default namespace
  - kubectl create ns {NAME}
  - kubectl config set-context --current --namespace={NAME}
- Create secret name by use docker-registry (secret docker-registry kubectl (when docker hub))
  - kubectl create secret docker-registry {Current Secret Name} --docker-server=https://index.docker.io/v1/ --docker-username={Username} --docker-password={Password} --docker-email={Email} -n {Namespace}
- Install Nginx Ingress (blog.saeloun.com/2023/03/21/setup-nginx-ingress-aws-eks)
  - helm repo add nginx-stable https://helm.nginx.com/stable
  - helm repo update
  - helm upgrade --install ingress-nginx ingress-nginx \
             --repo https://kubernetes.github.io/ingress-nginx \
             --namespace ingress-nginx --create-namespace
  - kubectl get pods -n ingress-nginx
- Install cert-manager (cert-manager.io/docs/installation/helm)
  - helm repo add jetstack https://charts.jetstack.io --force-update
  - helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.16.2 \
  --set crds.enabled=true
- Apply User deployment and service
  - kubectl apply -f user-template-deployment.yaml
- Apply User deployment and service
  - kubectl apply -f user-template-deployment.yaml
- Apply Ingress 
  - kubectl apply -f ingress-srv.yml
  - Warning: First apply ingress so next apply cert
- Apply Cert 
  - kubectl create -f windcard-cert.yml
  - Warning: First apply ingress so next apply cert

## Optional
- Change name when context name large
  - nano ~/.kube/config => change name: in - context
- Select or switch context name
  - kubectl config use-context {CONTEXT_NAME}
- Set and get Label node name (when not eks)
  - kubectl get nodes
  - kubectl label nodes {node-name} {key}={value}
  - kubectl get nodes --show-labels

## Optional AWS Role
- AWS EKS - role
  - service-role
    - AmazonEC2ContainerRegistryReadOnly
    - AmazonEKS_CNI_Policy
    - AmazonEKSClusterPolicy
    - AmazonEKSWorkerNodePolicy
    - AmazonSSMManagedInstanceCore
    - AmazonEKSVPCResourceController **
    - CloudWatchFullAccess **
    - eksctl-attractive-monster-1663593568-cluster-PolicyCloudWatchMetrics **
      ``` Policy
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Action": [
                      "cloudwatch:PutMetricData"
                  ],
                  "Resource": "*",
                  "Effect": "Allow"
              }
          ]
      }
      ```
    - eksctl-attractive-monster-1663593568-cluster-PolicyELBPermissions **
      ``` Policy
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Action": [
                      "ec2:DescribeAccountAttributes",
                      "ec2:DescribeAddresses",
                      "ec2:DescribeInternetGateways"
                  ],
                  "Resource": "*",
                  "Effect": "Allow"
              }
          ]
      }
      ```
  - node-group
    - AmazonEC2ContainerRegistryReadOnly
    - AmazonEKS_CNI_Policy
    - AmazonEKSWorkerNodePolicy
    - AmazonSSMManagedInstanceCore
    - certManagerDNS **
      ``` Policy
      {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "route53:GetChange",
                "Resource": "arn:aws:route53:::change/*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "route53:ChangeResourceRecordSets",
                    "route53:ListResourceRecordSets"
                ],
                "Resource": "arn:aws:route53:::hostedzone/*"
            },
            {
                "Effect": "Allow",
                "Action": "route53:ListHostedZonesByName",
                "Resource": "*"
            }
        ]
      }
      ```
  - For manage and action CI/CD
    - AmazonEBSCSIDriverPolicy
    - AmazonEC2ContainerRegistryFullAccess
    - AmazonEKS_CNI_Policy
    - AmazonEKSClusterPolicy
    - AmazonEKSVPCResourceController
    - AmazonEKSWorkerNodePolicy
    - AmazonSSMManagedInstanceCore
    - CloudWatchFullAccess
