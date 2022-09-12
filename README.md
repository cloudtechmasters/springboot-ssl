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

<img width="1132" alt="image" src="https://user-images.githubusercontent.com/68885738/189571727-4a6f44e0-0ce6-49dc-87c2-2c226d7f4b5e.png">


<img width="1219" alt="image" src="https://user-images.githubusercontent.com/68885738/189569606-57992408-a622-461b-a2e1-8c046a67b0d5.png">


<img width="1181" alt="image" src="https://user-images.githubusercontent.com/68885738/189540842-9a721908-a770-4476-a3e7-f1565fbe09e1.png">


<img width="1209" alt="image" src="https://user-images.githubusercontent.com/68885738/189540872-162a8a80-38b1-4579-bde8-2cdee3e26ef0.png">


<img width="1241" alt="image" src="https://user-images.githubusercontent.com/68885738/189540953-3da9d558-0022-4214-a879-1f7d318eb9bd.png">


<img width="569" alt="image" src="https://user-images.githubusercontent.com/68885738/189540817-f8febc31-e12a-4467-8793-46e595526e43.png">
