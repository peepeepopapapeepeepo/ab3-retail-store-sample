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
- Check Application Running

  ``` bash
  kubectl get pods -A
  ```

- Test access from Internet -> https://ab3.sawitmee.cc

### Change to use AWS PaaS without application change

- Let checkout use ElastiCache for Valkey 
  - Go to GitHub and run `02 - Deploy App on EKS`
    - Fill in `app` = `checkout-overrided`
    - Click `Run workflow`
  - Restart pod
      ``` bash
      kubectl -n checkout rollout restart deploy checkout
      ```
  - Check Application Running

      ``` bash
      kubectl -n checkout describe cm checkout
      kubectl -n checkout get po
      ```
  
  - Delete local Redis

      ``` bash
      kubectl -n checkout delete deploy checkout-redis
      kubectl -n checkout delete svc checkout-redis
      ``` 

- Let orders use Aurora for PostgreSQL
  - Go to GitHub and run `02 - Deploy App on EKS`
    - Fill in `app` =  `orders-overrided`
    - Click `Run workflow`
  - Restart pod

      ``` bash
      kubectl -n orders rollout restart deployment orders
      ```
      
  - Check Application Running

      ``` bash
      kubectl -n orders describe cm orders
      kubectl -n orders describe secretproviderclass
      kubectl -n orders get po
      ```

  - Delete local PostgreSQL

      ``` bash
      kubectl -n orders delete sts orders-postgresql
      kubectl -n orders delete svc orders-postgresql
      kubectl -n orders delete secret orders-db
      ```

- Test access from Internet -> https://ab3.sawitmee.cc

### Update Application and Deploy with GitHub

> Change banner from blue to green

- Edit `src/ui/src/main/resources/static/assets/css/styles.css`
- Go to GitHub and run `03 - Build Container Image`
  - Fill in 
    - `Application Name` =  `ui`
    - `Tag` = `0.8.5`
  - Click `Run workflow`
- Edit `deploy/ui/deployment.yaml`, change tag from `0.8.4` to `0.8.5`
- Deploy new version of `ui`
    - Go to GitHub and run `01 - Deploy Foundation App on EKS`
      - Fill in `Application Name` =  `ui`
      - Click `Kubernetes Manifest file name` = `deployment.yaml`
    - Check Application Running

        ``` bash
        kubectl -u ui get cm
        kubectl -u ui get po
        ```
- Test access from Internet -> https://ab3.sawitmee.cc

### Show EKS autoscaling by using load generator

- Show **HPA**

    ``` bash
    kubectl -n ui get hpa
    ```

- Show **Metric Server**

    ``` bash 
    kubectl -n kube-system get pod -l app.kubernetes.io/name=metrics-server
    ```

- Test metric is working 

    ``` bash
    kubectl top node
    kubectl top pods
    ```

- Run load generator inside EKS cluster

    ``` bash
    kubectl run load-generator \
    --image=williamyeh/hey:latest \
    --restart=Never -- -c 10 -q 5 -z 60m http://ui.ui.svc/home
    ```

- Monitor HPA

    ``` bash
    kubectl -n ui get hpa ui --watch
    ```

- Delete load generator

    ``` bash
    kubectl delete pod load-generator
    ```

### Show monitoring --> Container Insight

### Show CloudWatch Alarm

### Add Cache in CloudFront

- Walk through CloudFront cache behavior
- Run Terraform
- Test

### Show upgrade EKS