# springboot-ssl

Steps:
=======
1. Register a new DNS in AWS Route53
2. Request Certificate For Domain With In ACM
3. Create ALB-ingress-controller
4. Deploy springboot-app 
5. Create ingress for springboot using the certificate created in step-2


**1. Register a new DNS in AWS Route53**

Route 53 --> Hosted zones  --> Create hosted zone

<img width="869" alt="image" src="https://user-images.githubusercontent.com/68885738/189569781-39e41c8c-380e-482c-818e-50482b1801a6.png">

Click on create hosted zone.

<img width="1184" alt="image" src="https://user-images.githubusercontent.com/68885738/189569899-ea661c73-4152-457b-bfab-9b854daa6fd9.png">

Now copy the NS entried and create records in your domain provider. I am using namecheap.

<img width="1081" alt="image" src="https://user-images.githubusercontent.com/68885738/189570019-1b625c8e-6255-4fa5-981c-9601989d1f00.png">

Now you have DNS ready for your domain.

**2. Request Certificate For Domain With In ACM**

Goto secret manager --> ACM --> Request new certificate

<img width="1171" alt="image" src="https://user-images.githubusercontent.com/68885738/189571289-c4e93326-21bf-4f93-a47a-a94428f037ed.png">

<img width="1183" alt="image" src="https://user-images.githubusercontent.com/68885738/189571382-899e9e9f-14b6-4479-8487-c207b052a630.png">

Click on Request.

Now click on certificate, it will show in pending validation.

<img width="1436" alt="image" src="https://user-images.githubusercontent.com/68885738/189571465-518fd372-6f65-41c5-b7e0-9ed7265cbd5b.png">

Click on "Create DNS records in Amazon Route 53"

<img width="1182" alt="image" src="https://user-images.githubusercontent.com/68885738/189571555-677c57a8-29c2-4253-b2df-2fee0c6b4123.png">

In Route 53 you can see one cname record added automatically, after 10 minutes certificate status will become Issued.

<img width="1155" alt="image" src="https://user-images.githubusercontent.com/68885738/189570345-eeaa5fe2-7576-486a-b285-d756cd980806.png">

<img width="1450" alt="image" src="https://user-images.githubusercontent.com/68885738/189572113-19413f2a-e7fa-4595-9f83-428517df81ef.png">

Note down the certificate ARN.

**3. Create ALB-ingress-controller**

Create an OIDC provider and IAM role for the AWS Load Balancer Controller

a. Create an AWS Identity and Access Management (IAM) OIDC provider and associate the OIDC provider with your cluster:

     eksctl utils associate-iam-oidc-provider \
      --region us-east-1 \
      --cluster eksdemo-qa \
      --approve
      
b. Download an IAM policy for the AWS Load Balancer Controller:

     curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.0/docs/install/iam_policy.json

c. Create an IAM policy using the policy that you downloaded from step 2:

     aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

     {
         "Policy": {
             "PolicyName": "AWSLoadBalancerControllerIAMPolicy", 
             "PermissionsBoundaryUsageCount": 0, 
             "CreateDate": "2022-04-30T03:35:07Z", 
             "AttachmentCount": 0, 
             "IsAttachable": true, 
             "PolicyId": "ANPAWXLRIGN25YQXTQZDX", 
             "DefaultVersionId": "v1", 
             "Path": "/", 
             "Arn": "arn:aws:iam::4627568568669:policy/AWSLoadBalancerControllerIAMPolicy", 
             "UpdateDate": "2022-04-30T03:35:07Z"
         }
     }
     
Note: Copy the name of the policy's Amazon Resource Name (ARN) that's returned in Step 3.

d. Create an IAM role for the AWS Load Balancer Controller and attach the role to the service account created in step 2:

     eksctl create iamserviceaccount --cluster=eksdemo-qa --namespace=kube-system --name=aws-load-balancer-controller --attach-policy-arn=arn:aws:iam::4627568568669:policy/AWSLoadBalancerControllerIAMPolicy --override-existing-serviceaccounts --approve --region us-east-1
     
**Install the AWS Load Balancer Controller using Helm 3.0.0**

Install the TargetGroupBinding custom resource definitions:

     kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"
     customresourcedefinition.apiextensions.k8s.io/ingressclassparams.elbv2.k8s.aws created
     customresourcedefinition.apiextensions.k8s.io/targetgroupbindings.elbv2.k8s.aws created

Add the eks-charts repository:

     helm repo add eks https://aws.github.io/eks-charts

Install the AWS Load Balancer Controller using the command that corresponds to the AWS Region for your cluster:

     # helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller --set clusterName=eksdemo-qa --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller -n kube-system --set region=us-east-1


     Release "aws-load-balancer-controller" does not exist. Installing it now.
     NAME: aws-load-balancer-controller
     LAST DEPLOYED: Sat Apr 30 03:53:17 2022
     NAMESPACE: kube-system
     STATUS: deployed
     REVISION: 1
     TEST SUITE: None
     NOTES:
     AWS Load Balancer controller installed!
 
     [root@ip-172-31-25-208 ~]# helm ls -n kube-system
     NAME                        	NAMESPACE  	REVISION	UPDATED                               	STATUS  	CHART                             	APP VERSION
     aws-load-balancer-controller	kube-system	1       	2022-04-30 03:53:17.97800165 +0000 UTC	deployed	aws-load-balancer-controller-1.4.1	v2.4.1     

Verify that the AWS Load Balancer Controller is installed:

     # kubectl get deployment -n kube-system aws-load-balancer-controller
     NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
     aws-load-balancer-controller   2/2     2            2           47s
     
Now setup is done, whenever we create new ingress resource create it with below annotation:

     annotations:
         kubernetes.io/ingress.class: alb
         
Add either an internal or internet-facing annotation to specify where you want the Ingress to create your load balancer:

     alb.ingress.kubernetes.io/scheme: internal
     -or-

     alb.ingress.kubernetes.io/scheme: internet-facing

**4. Deploy springboot-app** 

     # kubectl apply -f deployement.yml
     deployment.apps/spring-boot-hello created

     # kubectl apply -f service.yml
     service/spring-boot-hello created

     # kubectl get pods
     NAME                                     READY   STATUS     RESTARTS   AGE
     spring-boot-hello-5c5fbbc5c4-wcgms       1/1     Running    0          27s

     # kubectl get svc
     NAME                                TYPE           CLUSTER-IP      EXTERNAL-IP                                           PORT(S)          AGE
     kubernetes                          ClusterIP      10.100.0.1      <none>                                                443/TCP          5h4m
     spring-boot-hello                   NodePort       10.100.32.241   <none>                                                8080:31412/TCP   30s

**5. Create ingress for springboot using the certificate created in step-2**

Add annotations related to SSL

     # SSL Setting - 1
         ## SSL Settings
         alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}, {"HTTP":80}]'
         alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:411686525067:certificate/8adf7812-a1af-4eae-af1b-ea425a238a67
         #alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-TLS-1-1-2017-01 #Optional (Picks default if not used)    
     # SSL Setting - 2
     spec:
       rules:
         #- host: ssldemo.tushar10pute.click    # SSL Setting (Optional only if we are not using certificate-arn annotation)

Deploy Ingress with SSL:

     # kubectl apply -f alb_ingress_ssl.yml

     # k get ing
     NAME                               CLASS    HOSTS                        ADDRESS   PORTS   AGE
     ingress-usermgmt-restapp-service   <none>   ssldemo.tushar10pute.click             80      25s

<img width="1132" alt="image" src="https://user-images.githubusercontent.com/68885738/189571727-4a6f44e0-0ce6-49dc-87c2-2c226d7f4b5e.png">


<img width="1219" alt="image" src="https://user-images.githubusercontent.com/68885738/189569606-57992408-a622-461b-a2e1-8c046a67b0d5.png">


<img width="1181" alt="image" src="https://user-images.githubusercontent.com/68885738/189540842-9a721908-a770-4476-a3e7-f1565fbe09e1.png">


<img width="1209" alt="image" src="https://user-images.githubusercontent.com/68885738/189540872-162a8a80-38b1-4579-bde8-2cdee3e26ef0.png">


<img width="1241" alt="image" src="https://user-images.githubusercontent.com/68885738/189540953-3da9d558-0022-4214-a879-1f7d318eb9bd.png">


<img width="569" alt="image" src="https://user-images.githubusercontent.com/68885738/189540817-f8febc31-e12a-4467-8793-46e595526e43.png">
