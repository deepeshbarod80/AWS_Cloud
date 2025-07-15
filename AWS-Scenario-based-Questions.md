
``` Scenario-Based Practice Questions for AWS Cloud Services (MERN App) ```

1. EKS Cluster Provisioning Failure
// Scenario: 
`Your Jenkins CD pipeline (Jenkinsfile.cd) uses Terraform to provision an EKS cluster for the MERN app, but the Terraform Apply stage fails with an error: "Error: creating EKS Cluster (mern-cluster): AccessDeniedException: User is not authorized to perform eks:CreateCluster." The pipeline uses AWS credentials (aws-credentials) stored in Jenkins.`

// Question: 
How would you troubleshoot and resolve this authorization issue to ensure the EKS cluster is created successfully?

// Answer: 
# Jenkins:
- I’d check the Jenkins console logs to pinpoint the `AccessDeniedException`

# AWS Credentials and Permissions:
- I'd confirm the IAM user or role tied to `aws-credentials`. The error suggests the IAM entity lacks `eks:CreateCluster` permission.
- In the AWS IAM Console, I’d verify the role’s policy, ensuring it includes `AmazonEKSClusterPolicy` or a custom policy with `eks:CreateCluster`, `eks:DescribeCluster`, and related actions.
- If missing, I’d attach the policy using 
```bash
aws iam attach-role-policy --role-name <role-name> --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
# To test locally with the same credentials, I’d run; 
aws eks create-cluster --name test-cluster 
```

- I’d also check for any SCPs or permission boundaries restricting EKS actions.
- If the issue persists, I’d validate the AWS region in Jenkinsfile.cd ("AWS_REGION = 'us-east-1') matches the Terraform config (terraform/modules/eks/main.tf).
- Finally, I’d re-run the pipeline and monitor logs.


---


2. ECR Image Push Failure
// Scenario:
`The Jenkins CI pipeline (Jenkinsfile.ci) builds and pushes MERN app Docker images to ECR, but the Push Docker Images stage fails with: "Error: denied: Your authorization token has expired. Reauthenticate and try again." The pipeline uses aws-credentials for authentication.`

// Question: 
What steps would you take to diagnose and fix this ECR authentication issue?

// Answer: The error indicates an expired ECR login token. 
- I’d verify the `aws ecr get-login-password` command in Jenkinsfile.ci is executed correctly within the withAWS block. 
- I’d check the `aws-credentials` in Jenkins to ensure the IAM user/role has `ecr:GetAuthorizationToken`, `ecr:BatchCheckLayerAvailability`, and `ecr:PutImage permissions`, updating the policy if needed (aws iam put-role-policy). 
- Next, I’d confirm the AWS CLI version on the Jenkins agent (aws --version) supports ECR authentication, upgrading if outdated. 
- To test, I’d manually run `aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com` and attempt a push. 
- If the issue persists, I’d check for IAM session timeouts or MFA requirements, ensuring the credentials are long-lived. I’d then re-run the pipeline.


---


3. CloudFront Serving Stale Content
// Scenario: 
`The MERN app’s frontend is served via CloudFront, configured in terraform/modules/cloudfront/main.tf with a default TTL of 86400 seconds. After a new deployment via ArgoCD, users see outdated content, and the X-Cache: Hit header confirms CloudFront is serving cached assets.`

// Question:
How would you resolve the issue of stale content and prevent it in future deployments?

// Answer: 
- To serve fresh content, I’d create a CloudFront invalidation for the affected paths: `aws cloudfront create-invalidation --distribution-id <dist-id> --paths "/*"`. 
- I’d verify the invalidation status in the AWS Console. 
- To prevent future issues, I’d update Jenkinsfile.cd to include an invalidation step after the `ArgoCD Sync` stage, using `aws cloudfront create-invalidation` with the distribution ID from Terraform outputs (`terraform/environments/dev/outputs.tf`). 
- I’d also review the frontend’s `Cache-Control` headers in `helm/mern-app/frontend/nginx.conf`, setting `max-age=0` or `no-cache` for dynamic content and `max-age=86400` for static assets like `JS/CSS`. 
- If needed, I’d lower the default TTL in `terraform/modules/cloudfront/main.tf` to `3600` seconds for faster cache refreshes. 
- Finally, I’d test by deploying a new image and checking the `X-Cache: Miss` header.


---


4. EKS Node Autoscaling Not Working
// Scenario: 
`The MERN app runs on EKS with an EC2 Auto Scaling Group (min=2, max=5) and the Cluster Autoscaler deployed via Helm (helm/cluster-autoscaler). During a spike in traffic, pods are pending, but no new nodes are added. Cluster Autoscaler logs show: "Scale-up not possible: no available subnets." `

// Question: 
Why is the Cluster Autoscaler failing to scale up, and how would you fix it?

// Answer: The error suggests the Auto Scaling Group can’t launch instances due to subnet issues. 
- I’d check `terraform/modules/eks/main.tf` to ensure the EKS node group’s `subnet_ids` include private subnets with sufficient IP addresses, as defined in `terraform/modules/vpc/main.tf`. 
- I’d verify `subnet CIDRs (10.0.3.0/24, 10.0.4.0/24)` have enough IPs for new EC2 instances (`aws ec2 describe-subnets`). 
- If IPs are exhausted, I’d expand the CIDR or add subnets in Terraform. Next, I’d confirm the `Auto Scaling Group’s tags (k8s.io/cluster-autoscaler/enabled=true)` match the Cluster Autoscaler config in `helm/cluster-autoscaler/values.yaml`. 
- I’d also check EC2 quotas in the AWS Console (`aws service-quotas list-services`) and request an increase if needed. 
- To test, I’d simulate load (`kubectl create -f stress-test.yaml`) and monitor node addition (`kubectl get nodes`). 
- If the issue persists, I’d redeploy the autoscaler if necessary.


---


5. ALB Ingress Not Routing Traffic
// Scenario: 
`The MERN app’s frontend and backend are exposed via an ALB created by an Ingress resource (helm/mern-app/templates/ingress.yaml).After deployment via ArgoCD, accessing "https://frontend.k8s.example.com" returns a 404 error, and ALB logs show no incoming requests.`

// Question: 
How would you troubleshoot and resolve the ALB routing issue?

// Answer: 
- I’d verify the Ingress resource is applied correctly: 
```bash
kubectl get ingress -n default 
```
- If present, I’d check the ALB’s target groups in the AWS Console to ensure the frontend pods (`app=mern-frontend`) are registered and healthy. 
- I’d inspect pod health and service configuration for confirming the selector matches pod labels;
```bash
kubectl get pods -l app=mern-frontend
kubectl describe pod -l app=mern-frontend
kubectl get svc mern-frontend-svc
```
- Next, I’d validate the Ingress annotations in `helm/mern-app/templates/ingress.yaml (e.g., alb.ingress.kubernetes.io/scheme: internet-facing)`
- I'd also ensure the `ACM certificate ARN` matches `terraform/modules/cloudfront/main.tf`. 
- I’d check `Route53 records` to confirm "frontend.k8s.example.com" points to the ALB;
```bash
aws route53 list-resource-record-sets --hosted-zone-id <zone-id> .
# If unresolved, I’d redeploy the Ingress
kubectl apply -f ingress.yaml
# monitor ALB access logs
aws logs filter-log-events
```
- Finally, I’d test with curl -v https://frontend.k8s.example.com.


---


6. SonarQube Analysis Failure
// Scenario: 
`The Jenkins CI pipeline’s SonarQube Analysis stage fails with: "ERROR: Not authorized. Please check the properties sonar.login and sonar.host.url." The pipeline uses sonarqube-token credentials and is configured to scan the MERN app’s frontend and backend code.`

// Question: 
What could be causing the authorization error, and how would you fix it?

// Answer: The error indicates an issue with the SonarQube token or server configuration. 
- I’d verify the sonarqube-token in Jenkins credentials is valid and matches a token generated in `SonarQube (http://sonarqube.<domain>:9000)`. 
- If invalid, I’d generate a new token in SonarQube’s User Settings and update Jenkins. Next, I’d check `Jenkinsfile.ci` to ensure the `withSonarQubeEnv` block correctly references the `SonarQube server` and `SONARQUBE_TOKEN`. 
- I’d confirm the `sonar.host.url (http://sonarqube.<domain>:9000)` is reachable from the `Jenkins agent (curl http://sonarqube.<domain>:9000)`. 
- If the URL is correct, I’d validate the `sonar-scanner` command’s parameters (`-Dsonar.projectKey=mern-x`, `-Dsonar.sources=helm/mern-app/frontend,helm/mern-app/backend`). 
- To test, I’d run sonar-scanner locally with the same token. If unresolved, I’d check SonarQube’s user permissions and re-run the pipeline.


---


7. Prometheus Not Scraping Metrics
// Scenario: 
`Prometheus is deployed in the EKS cluster’s monitoring namespace for the MERN app, but Grafana (https://grafana.k8s.example.com) shows no backend metrics. The Verify Monitoring stage in Jenkinsfile.cd passes, and Prometheus logs show: "Failed to scrape target: connection refused." `

// Question: 
How would you diagnose and fix the issue with Prometheus scraping?

// Answer: The “connection refused” error suggests Prometheus can’t reach the backend service. 
- I’d check the Prometheus service discovery targets in the UI (http://prometheus.<domain>:9090/targets) to confirm the mern-backend service is listed. 
- If missing, I’d verify the service monitor in `helm/mern-app/templates/prometheus-config.yaml` has correct labels (`prometheus.io/scrape: true`) then,
```bash
# I'd matche the backend service 
kubectl get svc mern-backend-svc 
# I’d ensure backend pods have Prometheus annotations in;
helm/mern-app/templates/backend-deployment.yaml
# Next, I’d check network policies to ensure Calico allows Prometheus to access backend ports;
kubernetes/network/calico-policy.yaml
# I’d test connectivity with commands;
kubectl port-forward -n <namespace> svc/mern-backend-svc 8080:8080 
curl localhost:8080/metrics 
# If unresolved, I’d restart Prometheus
kubectl rollout restart deployment prometheus -n monitoring
```
- I’d redeploy the Helm chart. I’d verify metrics in Grafana post-fix.


---


8. ArgoCD Application Out of Sync
// Scenario: 
`After a successful Jenkins CI pipeline run, the ArgoCD Sync stage in Jenkinsfile.cd completes, but the ArgoCD UI (https://argocd.k8s.example.com) shows the mern-app application as “Out of Sync” with a diff in the Helm values.yaml image tag.`

// Question: 
What could cause the sync failure, and how would you resolve it?
 
// Answer: The “Out of Sync” status indicates ArgoCD detects a mismatch between the Git repo and cluster state. 
- I’d check the ArgoCD UI for the diff, confirming the `image.tag` in `helm/mern-app/values.yaml` matches the `IMAGE_TAG` set in `Jenkinsfile.cd`. 
- If mismatched, I’d verify the Git commit was pushed successfully by Jenkins (`git log in helm/mern-app`). 
- Next, I’d run argocd app diff mern-app to inspect discrepancies and validate the Helm chart with helm template `helm/mern-app`. 
- If the image tag is correct, I’d check for `ArgoCD’s auto-sync policy` (`argocd app get mern-app`) and force a sync: `argocd app sync mern-app --force`. 
- I’d also ensure the argocd-token in Jenkins has sufficient permissions. 
- To prevent recurrence, I’d enable auto-sync in ArgoCD’s app manifest (`spec.syncPolicy.automated`). 
- Finally, I’d monitor the app status post-sync.


---


9. Trivy Scan Blocking Deployment
// Scenario: 
`The Trivy Image Scanning stage in Jenkinsfile.ci fails due to a HIGH severity vulnerability in the mern-mongodb image, blocking the pipeline. The Dockerfile (helm/mern-app/database/Dockerfile) uses mongo:4.4. `

// Question: 
How would you address the vulnerability to allow the pipeline to proceed?

// Answer: 
- I’d review the Trivy output in Jenkins to identify the vulnerable package (e.g., libssl1.1). 
- I’d update `helm/mern-app/database/Dockerfile` to use a newer `MongoDB image (mongo:5.0)` or install a `patched package (RUN apt-get update && apt-get install -y libssl1.1=1.1.1n)`. 
- I’d rebuild the image locally (`docker build -t test .`) and scan it (`trivy image test`) to confirm the fix. 
- I’d commit the updated Dockerfile to GitHub, triggering the CI pipeline. 
- To prevent future issues, I’d configure Dependabot in GitHub to update base images and enable ECR vulnerability scanning;
```bash
`aws ecr put-image-scanning-configuration --repository-name mern-mongodb --image-scanning-configuration scanOnPush=true
``` 
- Finally, II’d also review OWASP Dependency-Check reports in Jenkins artifacts for related vulnerabilities.


---


10. EKS Pod CrashLoopBackOff
// Scenario: 
`After deploying the MERN app’s backend via ArgoCD, some mern-backend pods enter a CrashLoopBackOff state. Logs (kubectl logs -l app=mern-backend) show: "MongoDB connection error: failed to connect to mongodb-service:27017." The MongoDB service and pods are running.`

// Question: 
How would you troubleshoot and resolve the pod crash issue?

// Answer: The error indicates the backend can’t connect to MongoDB. 
- I’d verify the MongoDB service (kubectl get svc mongodb-service) is reachable and matches the connection string in `helm/mern-app/values.yaml` (`e.g., mongodb://mongodb-service:27017`). 
- I’d check network policies (`kubernetes/network/calico-policy.yaml`) to ensure Calico allows traffic from mern-backend to mongodb-service on port 27017. 
- I’d test connectivity with kubectl exec -it <backend-pod> -- curl mongodb-service:27017. If the service is correct, 
- I’d inspect MongoDB pod logs (kubectl logs -l app=mern-mongodb) for errors and confirm it’s initialized. 
- I’d also validate the backend’s environment variables in `helm/mern-app/templates/backend-deployment.yaml` for correct MongoDB credentials (stored in Secrets Manager). 
- If needed, I’d redeploy with `helm upgrade mern-app helm/mern-app` and 
- I’d monitor pod status (kubectl get pods -l app=mern-backend).


---


