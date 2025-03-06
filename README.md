# POC | AnyCompany

## Existing Challenges

- Resource Over-Provisioning
- Traffic Scaling Bottleneck
- Heavy Day-2 Operations

## Procedure

### Walk through prepared infrastructure

- VPC
- EKS
- ECR
- ElastiCache for Valkey (1 Primary + 1 Replica)
- Aurora for PostgreSQL (1 Primary + 1 Replica)
- EC2
- Route 53
- ACM
- Web ACL + CloudFront

### Walk through IAM Role Federation to GitHub

- IAM Role
- GitHub Runner

### Deploy foundation application

- Go to GitHub and run `01 - Deploy Foundation App on EKS`
- Test access from Internet -> https://ab3.sawitmee.cc

### Change to use AWS PaaS without application change

- Let checkout use ElastiCache for Valkey
  - Go to GitHub and run `02 - Let Checkout use ElastiCache for Valkey`
  - Test access from Internet -> https://ab3.sawitmee.cc

- Let orders use Aurora for PostgreSQL
  - Go to GitHub and run `03 - Let Orders use Aurora for PostgreSQL`
  - Test access from Internet -> https://ab3.sawitmee.cc

### Update Application and Deploy with GitHub

> Change banner from blue to green

- Edit `src/ui/src/main/resources/static/assets/css/styles.css`
- Go to GitHub and run `04 - Build UI Container Image and Deploy`
- Test access from Internet -> https://ab3.sawitmee.cc

### Show EKS autoscaling by using load generator

- Show **Metric Server**

    ``` bash
    kubectl -n kube-system get pod -l app.kubernetes.io/name=metrics-server
    ```

- Test metric is working 

    ``` bash
    kubectl top node
    kubectl top pods
    ```

- Deploy HPA of **ui**
    - Go to GitHub and run `05 - Let UI use HPA`

- Run load generator inside EKS cluster

    ``` bash
    kubectl run load-generator \
    --image=williamyeh/hey:latest \
    --restart=Never -- -c 10 -q 5 -z 60m http://ui.ui.svc/home
    ```

- Monitor HPA

    ``` bash
    kubectl -n ui get hpa ui --watch
    watch kubectl -n ui get pods
    watch kubectl get nodes
    ```

- Delete load generator

    ``` bash
    kubectl delete pod load-generator
    ```

### Show monitoring

- https://console.aws.amazon.com/cloudwatch
  - Container Insight
  - Metrics
  - Logs

### Add Cache in CloudFront

- Enable cache for `/assets/*.jpg`
  - `terraform apply`
- Move static content to S3
  - `terraform apply`
  - Go to GitHub and run `06 - Delete Assets`