
### Application Load Balancer (ALB) in AWS

### Introduction
The **Application Load Balancer (ALB)** is a Layer 7 (application layer) load balancer provided by Amazon Web Services (AWS) as part of its Elastic Load Balancing (ELB) service. ALB operates at the HTTP/HTTPS level, making it ideal for web applications that require advanced routing capabilities, such as routing based on URL paths, hostnames, or HTTP headers. It distributes incoming application traffic across multiple targets, such as Amazon EC2 instances, containers, or Lambda functions, to improve availability, scalability, and fault tolerance.

### Role of ALB in AWS
ALB plays a critical role in modern cloud architectures by:
1. **Load Distribution**: Distributes incoming HTTP/HTTPS traffic across multiple healthy targets to ensure no single target is overwhelmed.
2. **High Availability**: Routes traffic to healthy instances, automatically replacing unhealthy ones, ensuring application uptime.
3. **Advanced Routing**: Supports content-based routing, allowing traffic to be directed based on URL paths, hostnames, or HTTP methods.
4. **Scalability**: Automatically scales to handle varying traffic loads, integrating seamlessly with Auto Scaling groups.
5. **Security**: Supports SSL/TLS termination, WebSocket, HTTP/2, and integration with AWS Web Application Firewall (WAF) for enhanced security.
6. **Serverless Integration**: Routes traffic to AWS Lambda functions, enabling serverless architectures.

### Important Concepts of ALB
1. **Listeners**: Rules that define how ALB processes incoming requests (e.g., HTTP on port 80 or HTTPS on port 443).
2. **Target Groups**: Groups of resources (e.g., EC2 instances, Lambda functions) to which ALB routes traffic. Each target group is associated with a health check to ensure only healthy targets receive traffic.
3. **Rules**: Conditions that determine how traffic is routed to target groups (e.g., based on URL path like `/api` or hostname).
4. **Health Checks**: ALB periodically checks the health of targets to ensure traffic is only routed to healthy ones.
5. **Sticky Sessions**: Session persistence that binds a user’s session to a specific target using cookies.
6. **SSL/TLS Termination**: ALB can handle SSL termination, offloading encryption/decryption from backend servers.
7. **WebSocket Support**: ALB supports persistent connections for real-time applications like chat or streaming.
8. **Cross-Zone Load Balancing**: Distributes traffic evenly across targets in all enabled Availability Zones (AZs).

### Steps to Create an ALB for a Real-World Project
Here’s a step-by-step guide to set up an ALB for a web application hosted on EC2 instances in a Virtual Private Cloud (VPC):

1. **Prepare Your Environment**:
   - Ensure you have a VPC with at least two subnets in different Availability Zones (public subnets for ALB).
   - Launch EC2 instances or containers (e.g., ECS) with your application running (e.g., a Node.js app on port 80).
   - Verify that your application is accessible and healthy.

2. **Create a Target Group**:
   - Go to the AWS Management Console > EC2 > Target Groups.
   - Click **Create Target Group**.
   - Specify:
     - **Target Type**: Instance, IP, or Lambda.
     - **Name**: e.g., `my-app-target-group`.
     - **Protocol/Port**: HTTP/80 (or as per your app).
     - **VPC**: Select your VPC.
   - Configure health checks (e.g., path `/health`, HTTP 200 response).
   - Register your EC2 instances or other targets to the group.

3. **Create the ALB**:
   - In the EC2 Console, go to **Load Balancers** > **Create Load Balancer** > **Application Load Balancer**.
   - Configure:
     - **Name**: e.g., `my-app-alb`.
     - **Scheme**: Internet-facing (for public apps) or internal.
     - **Listeners**: Add HTTP (port 80) and/or HTTPS (port 443).
     - **Availability Zones**: Select at least two subnets in different AZs.
     - **Security Groups**: Create or assign a security group allowing inbound traffic on ports 80/443.
     - For HTTPS, upload an SSL certificate via AWS Certificate Manager (ACM) or IAM.

4. **Configure Listener Rules**:
   - Add rules to the listener (e.g., forward `/api/*` to `api-target-group` and `/web/*` to `web-target-group`).
   - Set a default rule to forward traffic to a primary target group.

5. **Test the ALB**:
   - Copy the ALB’s DNS name (e.g., `my-app-alb-123456789.us-east-1.elb.amazonaws.com`).
   - Access it via a browser or `curl` to verify routing and health checks.
   - Simulate instance failure to ensure ALB reroutes traffic to healthy instances.

6. **Integrate with Other AWS Services** (Optional):
   - Enable AWS WAF for security.
   - Configure Route 53 to map a custom domain (e.g., `app.example.com`) to the ALB.
   - Enable access logs in an S3 bucket for monitoring.

7. **Monitor and Scale**:
   - Use CloudWatch to monitor ALB metrics (e.g., request count, latency, HTTP errors).
   - Integrate with Auto Scaling to dynamically adjust the number of instances based on traffic.

### Challenges in Implementing ALB
1. **Complex Routing Rules**:
   - Misconfigured rules (e.g., incorrect path patterns) can lead to routing errors or downtime.
   - **Solution**: Test rules thoroughly and use default rules as a fallback.

2. **SSL/TLS Configuration**:
   - Managing SSL certificates for HTTPS can be challenging, especially for multiple domains.
   - **Solution**: Use AWS Certificate Manager (ACM) for automated certificate management.

3. **Health Check Failures**:
   - Incorrect health check settings (e.g., wrong path or timeout) can mark healthy instances as unhealthy.
   - **Solution**: Validate health check endpoints and adjust thresholds (e.g., timeout, interval).

4. **Cost Management**:
   - ALB pricing is based on Load Balancer Capacity Units (LCUs), which can increase with high traffic.
   - **Solution**: Monitor usage via CloudWatch and optimize target group size.

5. **Cross-Zone Load Balancing Overhead**:
   - Enabling cross-zone load balancing can incur data transfer costs between AZs.
   - **Solution**: Evaluate if cross-zone balancing is necessary or use zonal isolation for cost efficiency.

6. **Latency for WebSocket Applications**:
   - Improper configuration for WebSocket traffic can lead to connection drops.
   - **Solution**: Ensure WebSocket protocols are enabled and timeout settings are adjusted.

### Factors Affecting ALB Performance and Working
1. **Traffic Volume**:
   - High request rates or large payloads can increase latency or LCU costs.
   - **Mitigation**: Optimize application performance and enable caching (e.g., via CloudFront).

2. **Target Health**:
   - Unhealthy targets reduce the pool of available instances, increasing load on others.
   - **Mitigation**: Regularly monitor health checks and fix application issues promptly.

3. **Network Configuration**:
   - Misconfigured security groups, subnets, or route tables can block traffic to the ALB or targets.
   - **Mitigation**: Double-check VPC, subnet, and security group settings.

4. **Listener Rules Complexity**:
   - Too many rules or overly complex conditions can slow down request processing.
   - **Mitigation**: Simplify rules and prioritize frequently used paths.

5. **Backend Application Performance**:
   - Slow or resource-intensive backend services can bottleneck ALB performance.
   - **Mitigation**: Optimize application code, database queries, or use Auto Scaling.

6. **SSL/TLS Overhead**:
   - SSL termination at the ALB can introduce latency for high-traffic HTTPS applications.
   - **Mitigation**: Offload static content to a CDN or use HTTP/2 for better performance.

7. **Region and AZ Selection**:
   - Choosing AZs with high latency or limited capacity can degrade performance.
   - **Mitigation**: Select AZs with low latency and enable cross-zone load balancing if needed.

8. **Monitoring and Logging**:
   - Lack of proper monitoring can delay detection of performance issues.
   - **Mitigation**: Enable CloudWatch metrics and ALB access logs for proactive monitoring.

### Additional Notes
- For real-world projects, consider using **Infrastructure as Code (IaC)** tools like AWS CloudFormation or Terraform to automate ALB setup and reduce human error.
- Regularly review AWS Trusted Advisor recommendations for ALB optimization.
- If you need real-time insights or specific examples from recent X posts or web content about ALB implementations, I can search for those upon request.


---


# ALB Troubleshooting

Troubleshooting Application Load Balancer (ALB) issues in a real-world microservices project using Amazon Elastic Kubernetes Service (EKS) can be complex due to the integration of ALB with Kubernetes, microservices, and AWS infrastructure. Below is a detailed guide addressing common ALB issues faced by DevOps engineers when implementing ALB with EKS, along with their causes, symptoms, and resolution steps. The focus is on practical troubleshooting steps tailored for a microservices architecture.


## **Troubleshooting ALB Issues in EKS for Microservices Projects**

This guide provides detailed steps to diagnose and resolve common Application Load Balancer (ALB) issues when used with Amazon Elastic Kubernetes Service (EKS) in a microservices project. Each issue includes symptoms, potential causes, and resolution steps, tailored for DevOps engineers managing microservices deployments.

## Issue 1: ALB Not Routing Traffic to Pods
**Symptoms**:
- HTTP 502/503 errors when accessing the ALB DNS name.
- ALB health checks report targets as unhealthy.
- No traffic reaches the Kubernetes pods.

**Potential Causes**:
1. **Misconfigured Target Groups**:
   - Pods are not registered correctly with the ALB target group.
   - Incorrect health check settings (e.g., wrong path or port).
2. **AWS ALB Ingress Controller Issues**:
   - The AWS ALB Ingress Controller is not properly installed or configured.
   - Ingress resource annotations are incorrect.
3. **Security Group Misconfiguration**:
   - ALB or EKS worker node security groups block traffic.
4. **Pod Health Issues**:
   - Application in the pod is not responding to health checks.
5. **Service Misconfiguration**:
   - Kubernetes Service does not match the ALB target group port or selector.

**Resolution Steps**:
1. **Verify Target Group Registration**:
   - Go to AWS Console > EC2 > Target Groups.
   - Check if the correct pods or node IPs are registered.
   - Ensure the target group port matches the Kubernetes Service port (e.g., `80` or `8080`).
2. **Check Health Check Settings**:
   - Verify the health check path (e.g., `/health`) returns HTTP 200.
   - Adjust timeout, interval, and thresholds in the target group settings if needed.
   - Example: If the app’s health endpoint is `/api/health`, update the target group health check path accordingly.
3. **Validate ALB Ingress Controller**:
   - Ensure the AWS Load Balancer Controller is installed in the EKS cluster:
     ```bash
     kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
     ```
   - Check controller logs for errors:
     ```bash
     kubectl logs -n kube-system <controller-pod-name>
     ```
   - Verify Ingress annotations in your Kubernetes manifest:
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: my-app-ingress
       namespace: my-namespace
       annotations:
         kubernetes.io/ingress.class: alb
         alb.ingress.kubernetes.io/scheme: internet-facing
         alb.ingress.kubernetes.io/target-type: instance
         alb.ingress.kubernetes.io/healthcheck-path: /health
     spec:
       rules:
       - http:
           paths:
           - path: /api
             pathType: Prefix
             backend:
               service:
                 name: my-service
                 port:
                   number: 80
     ```
   - Ensure the `alb.ingress.kubernetes.io` annotations are correct.
4. **Inspect Security Groups**:
   - Ensure the ALB security group allows inbound traffic on ports `80`/`443` from the internet or VPC CIDR.
   - Ensure the EKS worker node security group allows inbound traffic from the ALB security group on the pod’s port.
   - Use AWS CLI to verify:
     ```bash
     aws ec2 describe-security-groups --group-ids <alb-security-group-id>
     ```
5. **Check Pod Health**:
   - Exec into a pod and test the health endpoint:
     ```bash
     kubectl exec -it <pod-name> -n <namespace> -- curl http://localhost:8080/health
     ```
   - Fix application code if the endpoint returns errors.
6. **Validate Kubernetes Service**:
   - Ensure the Service selector matches pod labels:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: my-service
       namespace: my-namespace
     spec:
       selector:
         app: my-app
       ports:
       - port: 80
         targetPort: 8080
     ```
   - Verify Service is running:
     ```bash
     kubectl get svc -n <namespace>
     ```

## Issue 2: 502 Bad Gateway Errors
**Symptoms**:
- ALB returns HTTP 502 errors when accessing the application.
- CloudWatch metrics show `HTTPCode_ELB_5XX` errors.

**Potential Causes**:
1. **Backend Application Failure**:
   - The application in the pod returns invalid responses (e.g., malformed HTTP headers).
2. **Mismatched Protocols**:
   - ALB listener (e.g., HTTPS) does not match the backend Service protocol (e.g., HTTP).
3. **Idle Timeout Exceeded**:
   - Backend pods take too long to respond, exceeding ALB’s idle timeout.
4. **Pod Crashes or Resource Limits**:
   - Pods are crashing due to insufficient CPU/memory or application errors.

**Resolution Steps**:
1. **Check Application Logs**:
   - View pod logs to identify errors:
     ```bash
     kubectl logs <pod-name> -n <namespace>
     ```
   - Fix application code if it’s returning invalid responses.
2. **Verify Protocol Consistency**:
   - Ensure the ALB listener protocol (e.g., HTTPS on 443) matches the backend Service protocol.
   - Update Ingress annotations if needed:
     ```yaml
     alb.ingress.kubernetes.io/backend-protocol: HTTP
     ```
3. **Adjust Idle Timeout**:
   - Increase the ALB idle timeout (default: 60 seconds) if the application requires longer processing:
     - Go to AWS Console > EC2 > Load Balancers > Attributes > Edit > Idle Timeout.
     - Or update via AWS CLI:
       ```bash
       aws elbv2 modify-load-balancer-attributes --load-balancer-arn <alb-arn> --attributes Key=idle_timeout.timeout_seconds,Value=120
       ```
4. **Inspect Pod Resources**:
   - Check if pods are crashing due to resource limits:
     ```bash
     kubectl describe pod <pod-name> -n <namespace>
     ```
   - Increase CPU/memory limits in the pod spec if needed:
     ```yaml
     resources:
       limits:
         cpu: "1"
         memory: "512Mi"
       requests:
         cpu: "0.5"
         memory: "256Mi"
     ```

## Issue 3: Traffic Not Distributed Evenly Across Pods
**Symptoms**:
- Some pods receive significantly more traffic than others.
- Uneven CPU/memory utilization across nodes/pods.

**Potential Causes**:
1. **Session Persistence Enabled**:
   - Sticky sessions are enabled, causing traffic to stick to specific pods.
2. **Improper Load Balancing Algorithm**:
   - ALB is not configured to use round-robin or least outstanding requests.
3. **Node Affinity or Taints**:
   - Kubernetes pod scheduling causes uneven distribution across nodes.
4. **Cross-Zone Load Balancing Disabled**:
   - Traffic is not distributed across Availability Zones (AZs).

**Resolution Steps**:
1. **Disable Sticky Sessions** (if not required):
   - Check Ingress annotations for sticky sessions:
     ```yaml
     alb.ingress.kubernetes.io/affinity: cookie
     ```
   - Remove or disable if not needed:
     ```bash
     kubectl edit ingress my-app-ingress -n <namespace>
     ```
2. **Verify Load Balancing Algorithm**:
   - ALB uses round-robin by default. Confirm no custom settings are overriding this.
   - Check target group attributes:
     ```bash
     aws elbv2 describe-target-group-attributes --target-group-arn <target-group-arn>
     ```
3. **Check Pod Scheduling**:
   - Verify pod distribution across nodes:
     ```bash
     kubectl get pods -o wide -n <namespace>
     ```
   - Adjust pod anti-affinity rules to spread pods across nodes:
     ```yaml
     affinity:
       podAntiAffinity:
         preferredDuringSchedulingIgnoredDuringExecution:
         - weight: 100
           podAffinityTerm:
             labelSelector:
               matchLabels:
                 app: my-app
             topologyKey: kubernetes.io/hostname
     ```
4. **Enable Cross-Zone Load Balancing**:
   - Ensure cross-zone load balancing is enabled for even distribution across AZs:
     ```bash
     aws elbv2 modify-load-balancer-attributes --load-balancer-arn <alb-arn> --attributes Key=load_balancing.cross_zone.enabled,Value=true
     ```

## Issue 4: ALB DNS Not Resolving or Inaccessible
**Symptoms**:
- ALB DNS name (e.g., `my-alb-123456789.us-east-1.elb.amazonaws.com`) does not resolve.
- Timeout or connection refused errors when accessing the ALB.

**Potential Causes**:
1. **Route 53 or DNS Misconfiguration**:
   - Custom domain not properly mapped to ALB via Route 53.
2. **ALB Not Provisioned Correctly**:
   - Ingress controller failed to create the ALB.
3. **VPC or Subnet Issues**:
   - Subnets lack proper route tables or internet gateway for internet-facing ALB.
4. **Security Group Restrictions**:
   - ALB security group blocks inbound traffic.

**Resolution Steps**:
1. **Verify Route 53 Configuration**:
   - Check if the custom domain (e.g., `app.example.com`) is mapped to the ALB via an Alias record in Route 53:
     ```bash
     aws route53 list-resource-record-sets --hosted-zone-id <zone-id>
     ```
   - Create or update the Alias record to point to the ALB DNS name.
2. **Check ALB Provisioning**:
   - Verify the ALB exists in the AWS Console > EC2 > Load Balancers.
   - Check Ingress controller logs for provisioning errors:
     ```bash
     kubectl logs -n kube-system <controller-pod-name>
     ```
   - Ensure the Ingress resource has the correct `ingressClassName` or annotation.
3. **Validate VPC and Subnets**:
   - Ensure the ALB is deployed in public subnets with an internet gateway.
   - Verify route tables have a `0.0.0.0/0` route to the internet gateway:
     ```bash
     aws ec2 describe-route-tables --route-table-id <route-table-id>
     ```
4. **Inspect Security Groups**:
   - Ensure the ALB security group allows inbound traffic on `80`/`443` from `0.0.0.0/0` (or your CIDR).
   - Update rules if needed:
     ```bash
     aws ec2 authorize-security-group-ingress --group-id <alb-security-group-id> --protocol tcp --port 80 --cidr 0.0.0.0/0
     ```

## Issue 5: High Latency or Slow Response Times
**Symptoms**:
- Slow response times when accessing the application via ALB.
- CloudWatch metrics show high `TargetResponseTime`.

**Potential Causes**:
1. **Backend Application Performance**:
   - Microservices have slow database queries or inefficient code.
2. **Insufficient Resources**:
   - Pods or nodes are under-provisioned for the traffic load.
3. **ALB Configuration**:
   - Improper listener or target group settings (e.g., no HTTP/2 support).
4. **Network Latency**:
   - ALB and pods are in different AZs, causing inter-AZ latency.

**Resolution Steps**:
1. **Profile Application Performance**:
   - Use application performance monitoring (APM) tools like AWS X-Ray to identify bottlenecks.
   - Optimize slow database queries or add caching (e.g., Redis, ElastiCache).
2. **Scale Resources**:
   - Increase pod replicas or node capacity:
     ```bash
     kubectl scale deployment my-app -n <namespace> --replicas=5
     ```
   - Enable Cluster Autoscaler for EKS to dynamically scale nodes.
3. **Optimize ALB Settings**:
   - Enable HTTP/2 for better performance:
     ```yaml
     alb.ingress.kubernetes.io/protocol-version: HTTP2
     ```
   - Adjust deregistration delay to remove unhealthy targets faster:
     ```bash
     aws elbv2 modify-target-group-attributes --target-group-arn <target-group-arn> --attributes Key=deregistration_delay.timeout_seconds,Value=30
     ```
4. **Minimize Inter-AZ Latency**:
   - Deploy ALB and worker nodes in the same AZs if cross-zone load balancing is not required.
   - Check pod placement:
     ```bash
     kubectl get pods -o wide -n <namespace>
     ```

## General Troubleshooting Tips
- **Enable ALB Access Logs**:
  - Configure access logs to an S3 bucket to analyze request patterns and errors:
    ```bash
    aws elbv2 modify-load-balancer-attributes --load-balancer-arn <alb-arn> --attributes Key=access_logs.s3.enabled,Value=true Key=access_logs.s3.bucket,Value=<bucket-name> Key=access_logs.s3.prefix,Value=<prefix>
    ```
- **Monitor CloudWatch Metrics**:
  - Key metrics to monitor: `RequestCount`, `TargetResponseTime`, `HTTPCode_ELB_5XX`, `HealthyHostCount`.
  - Set up CloudWatch alarms for proactive alerts.
- **Use AWS CLI or kubectl for Debugging**:
  - Query ALB details:
    ```bash
    aws elbv2 describe-load-balancers --load-balancer-arns <alb-arn>
    ```
  - Check Ingress status:
    ```bash
    kubectl describe ingress my-app-ingress -n <namespace>
    ```
- **Test Incrementally**:
  - Test pod connectivity locally before exposing via ALB:
    ```bash
    kubectl port-forward svc/my-service -n <namespace> 8080:80
    ```

## Example: Debugging a 502 Error
1. Check ALB access logs in S3 for detailed error codes.
2. Verify the pod’s health endpoint is responding correctly:
   ```bash
   kubectl exec -it <pod-name> -n <namespace> -- curl http://localhost:8080/health
   ```
3. If the pod is healthy but ALB reports 502, check for protocol mismatches in the Ingress annotations.
4. Ensure the EKS worker node security group allows traffic from the ALB.



### Additional Notes
- **Automation**: Use tools like Terraform or Helm to manage ALB and Ingress configurations to reduce manual errors. For example, the AWS Load Balancer Controller can be deployed via Helm:
  ```bash
  helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system
  ```
- **Testing**: Simulate traffic using tools like `k6` or `Locust` to identify performance bottlenecks before going to production.
- **Documentation**: Refer to the [AWS Load Balancer Controller documentation](https://kubernetes-sigs.github.io/aws-load-balancer-controller/) for advanced configurations.


---


## Recent Issues with ALB in EKS for Microservices Projects

### 1. HTTP 503 Service Unavailable Errors
**Reported**: July 2025 (X posts by @ITNEXT_io and @ITNEXT_DevOps)
**Symptoms**:
- Users receive HTTP 503 errors when accessing the ALB DNS name.
- CloudWatch metrics show elevated `HTTPCode_ELB_5XX` errors.
- Traffic does not reach the intended pods, or the ALB reports no healthy targets.

**Potential Causes**:
1. **Misconfigured Health Checks**:
   - The health check path (e.g., `/health`) is incorrect or the application does not return HTTP 200.
   - Health check timeouts or thresholds are too strict.
2. **AWS Load Balancer Controller Issues**:
   - Incorrect Ingress annotations or outdated controller version causing improper ALB provisioning.
3. **Pod Readiness Issues**:
   - Pods fail readiness probes, causing them to be marked unhealthy by the ALB.
4. **Subnet or Security Group Misconfiguration**:
   - Subnets lack proper tags (`kubernetes.io/role/elb` or `kubernetes.io/role/internal-elb`).
   - Security groups block traffic between ALB and EKS worker nodes.

**Troubleshooting Steps**:
1. **Verify Health Checks**:
   - Check the target group’s health check settings in the AWS Console > EC2 > Target Groups.
   - Ensure the health check path (e.g., `/health`) is correct and returns HTTP 200:
     ```bash
     kubectl exec -it <pod-name> -n <namespace> -- curl http://localhost:8080/health
     ```
   - Adjust health check settings (e.g., increase timeout or threshold) if needed:
     ```bash
     aws elbv2 modify-target-group --target-group-arn <target-group-arn> --health-check-timeout-seconds 10 --healthy-threshold-count 3
     ```
2. **Check AWS Load Balancer Controller**:
   - Verify the controller version (recommended: 2.4.4 or later):[](https://repost.aws/knowledge-center/load-balancer-troubleshoot-creating)
     ```bash
     kubectl get deployment -n kube-system aws-load-balancer-controller -o yaml | grep image
     ```
   - Update to the latest version using Helm if outdated:
     ```bash
     helm upgrade aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<your-cluster-name>
     ```
   - Check controller logs for errors:
     ```bash
     kubectl logs -n kube-system <controller-pod-name>
     ```
   - Validate Ingress annotations:
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: my-app-ingress
       namespace: my-namespace
       annotations:
         kubernetes.io/ingress.class: alb
         alb.ingress.kubernetes.io/scheme: internet-facing
         alb.ingress.kubernetes.io/target-type: ip
         alb.ingress.kubernetes.io/healthcheck-path: /health
     spec:
       rules:
       - http:
           paths:
           - path: /api
             pathType: Prefix
             backend:
               service:
                 name: my-service
                 port:
                   number: 80
     ```
3. **Validate Pod Readiness**:
   - Check pod status and readiness probes:
     ```bash
     kubectl describe pod <pod-name> -n <namespace>
     ```
   - Ensure readiness probes are correctly configured in the pod spec:
     ```yaml
     readinessProbe:
       httpGet:
         path: /health
         port: 8080
       initialDelaySeconds: 5
       periodSeconds: 10
     ```
   - Fix application issues if the probe fails.
4. **Verify Subnet and Security Group Configuration**:
   - Ensure subnets are tagged correctly for ALB discovery:[](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)
     ```bash
     aws ec2 create-tags --resources <subnet-id> --tags Key=kubernetes.io/role/elb,Value=1
     ```
   - Confirm at least two subnets in different Availability Zones are available.[](https://repost.aws/knowledge-center/load-balancer-troubleshoot-creating)
   - Check security groups:
     - ALB security group should allow inbound `80`/`443` from the internet or VPC CIDR.
     - EKS worker node security group should allow inbound traffic from the ALB on the pod’s port.
     ```bash
     aws ec2 describe-security-groups --group-ids <alb-security-group-id>
     ```

### 2. gRPC Support Issues with ALB
**Reported**: July 2025 (X post by @0x15f)
**Symptoms**:
- gRPC services fail to work with ALB when using service-to-service communication with local certificates.
- Connection drops or errors occur when ALB is configured for gRPC traffic without SSL.

**Potential Causes**:
- ALB’s gRPC support requires SSL/TLS for out-of-the-box functionality, but local certificate setups (e.g., self-signed certs for internal services) are not supported by default.
- Missing or incorrect ALB listener configuration for gRPC (e.g., HTTP/2 not enabled).
- Timeout settings not optimized for long-lived gRPC streams.

**Troubleshooting Steps**:
1. **Enable HTTP/2 for gRPC**:
   - Ensure the ALB listener is configured for HTTP/2, which is required for gRPC:
     ```yaml
     alb.ingress.kubernetes.io/protocol-version: HTTP2
     ```
   - Update the listener in the AWS Console or via CLI:
     ```bash
     aws elbv2 modify-listener --listener-arn <listener-arn> --protocol HTTP --port 443
     ```
2. **Configure SSL for gRPC**:
   - If using local certificates, set up a custom SSL certificate in AWS Certificate Manager (ACM) or upload it to IAM.
   - Update the Ingress to use SSL:
     ```yaml
     alb.ingress.kubernetes.io/certificate-arn: <certificate-arn>
     alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-TLS-1-2-2017-01
     ```
   - For internal services, consider using mutual TLS (mTLS) and configure the ALB to trust the local CA.
3. **Adjust Timeout for gRPC Streams**:
   - Increase the ALB idle timeout to accommodate long-lived gRPC connections:
     ```bash
     aws elbv2 modify-load-balancer-attributes --load-balancer-arn <alb-arn> --attributes Key=idle_timeout.timeout_seconds,Value=3600
     ```
4. **Test gRPC Connectivity**:
   - Use a gRPC client to test connectivity to the ALB DNS name.
   - If issues persist, consider using a Network Load Balancer (NLB) for gRPC, as it operates at Layer 4 and may bypass some ALB limitations for non-SSL gRPC traffic.[](https://docs.aws.amazon.com/eks/latest/best-practices/load-balancing.html)

### 3. ALB Not Provisioned After Creating Ingress
**Reported**: AWS re:Post and documentation (2023, still relevant in 2025)[](https://repost.aws/knowledge-center/load-balancer-troubleshoot-creating)[](https://repost.aws/knowledge-center/eks-alb-ingress-controller-setup)
**Symptoms**:
- Ingress resource is created, but no ALB is provisioned in the AWS Console.
- `kubectl describe ingress` shows errors like “invalid ingress class” or “controller failed to reconcile.”

**Potential Causes**:
1. **AWS Load Balancer Controller Not Installed or Misconfigured**:
   - The controller is not running or lacks proper IAM permissions.
2. **Invalid IngressClass or Annotations**:
   - Missing `kubernetes.io/ingress.class: alb` or incorrect `ingressClassName`.
3. **Subnet Tagging Issues**:
   - Subnets lack required tags for auto-discovery (`kubernetes.io/role/elb` or `kubernetes.io/role/internal-elb`).
4. **IAM Role Issues**:
   - The controller’s IAM role lacks permissions to create ALB resources.

**Troubleshooting Steps**:
1. **Verify Controller Installation**:
   - Check if the AWS Load Balancer Controller is running:
     ```bash
     kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
     ```
   - Install or upgrade the controller using Helm if needed:[](https://medium.com/%40shivam77kushwah/integrate-application-load-balancer-with-aws-eks-using-aws-load-balancer-controller-7e9b7d178a79)
     ```bash
     helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<your-cluster-name>
     ```
2. **Check IngressClass**:
   - Ensure the Ingress specifies the correct class:
     ```yaml
     spec:
       ingressClassName: alb
     ```
   - Verify the IngressClass resource exists:
     ```bash
     kubectl get ingressclass alb
     ```
   - If missing, create it:
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: IngressClass
     metadata:
       name: alb
     spec:
       controller: ingress.k8s.aws/alb
     ```
3. **Validate Subnet Tags**:
   - Ensure subnets are tagged for ALB discovery:[](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)
     ```bash
     aws ec2 describe-subnets --filters Name=vpc-id,Values=<vpc-id> --query "Subnets[*].{ID:SubnetId,Tags:Tags}"
     ```
   - Add tags if missing:
     ```bash
     aws ec2 create-tags --resources <subnet-id> --tags Key=kubernetes.io/role/elb,Value=1
     ```
4. **Check IAM Permissions**:
   - Ensure the controller’s IAM role has the `AWSLoadBalancerControllerIAMPolicy` attached:[](https://repost.aws/knowledge-center/eks-alb-ingress-controller-setup)
     ```bash
     aws iam attach-role-policy --policy-arn arn:aws:iam::<account-id>:policy/AWSLoadBalancerControllerIAMPolicy --role-name <controller-role-name>
     ```
   - Use IAM Roles for Service Accounts (IRSA) for better security:
     ```yaml
     metadata:
       annotations:
         eks.amazonaws.com/role-arn: arn:aws:iam::<account-id>:role/aws-load-balancer-controller
     ```

### 4. Exceeding Security Group Rule Quotas
**Reported**: AWS documentation (2023, relevant in 2025)[](https://docs.aws.amazon.com/eks/latest/userguide/network-load-balancing.html)[](https://repost.aws/knowledge-center/load-balancer-troubleshoot-creating)
**Symptoms**:
- ALB provisioning fails with errors about exceeding security group rule limits.
- `kubectl describe ingress` or controller logs show errors related to security group updates.

**Potential Causes**:
- EKS attempts to add inbound rules for ALB health checks and client traffic, exceeding the default quota (60 rules per security group).
- Multiple ALBs or services in the cluster increase the number of rules.

**Troubleshooting Steps**:
1. **Check Security Group Rules**:
   - List rules for the EKS worker node security group:
     ```bash
     aws ec2 describe-security-groups --group-ids <node-security-group-id>
     ```
   - Identify the number of rules and their sources.
2. **Request Quota Increase**:
   - Request an increase in the “Rules per security group” quota via AWS Service Quotas:
     ```bash
     aws service-quotas request-service-quota-increase --service-code vpc --quota-code L-0B573B4E --desired-value 100
     ```
3. **Optimize Security Group Rules**:
   - Consolidate rules by using CIDR ranges instead of individual IPs.
   - Remove unused ALBs or services to reduce rule count.
4. **Use Instance Targets Instead of IP Targets**:
   - Switch to instance-based targets to reduce the number of rules (IP targets create a rule per pod IP):[](https://aws.amazon.com/blogs/containers/how-to-leverage-application-load-balancers-advanced-request-routing-to-route-application-traffic-across-multiple-amazon-eks-clusters/)
     ```yaml
     alb.ingress.kubernetes.io/target-type: instance
     ```

### 5. Slow ALB Provisioning or Updates
**Reported**: General DevOps feedback and AWS documentation[](https://aws.amazon.com/blogs/containers/how-to-leverage-application-load-balancers-advanced-request-routing-to-route-application-traffic-across-multiple-amazon-eks-clusters/)
**Symptoms**:
- ALB takes several minutes to provision or update after creating/modifying an Ingress.
- Microservices experience delays in becoming available.

**Potential Causes**:
- Large number of Ingress rules or target groups slows down controller reconciliation.
- Insufficient IP addresses in subnets cause provisioning delays.[](https://repost.aws/knowledge-center/eks-load-balancers-troubleshooting)
- Controller is overloaded due to high cluster activity.

**Troubleshooting Steps**:
1. **Check Subnet IP Availability**:
   - Ensure each subnet has at least eight free IP addresses:[](https://repost.aws/knowledge-center/eks-load-balancers-troubleshooting)
     ```bash
     aws ec2 describe-subnets --subnet-ids <subnet-id> --query "Subnets[*].AvailableIpAddressCount"
     ```
   - Add more subnets or increase subnet CIDR range if needed.
2. **Optimize Ingress Rules**:
   - Use IngressGroups to share a single ALB across multiple services, reducing the number of ALBs:[](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)
     ```yaml
     alb.ingress.kubernetes.io/group.name: my-app-group
     ```
   - Limit the number of rules per Ingress (63 or fewer characters for group names).[](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)
3. **Scale Controller Resources**:
   - Increase the controller’s resource limits:
     ```yaml
     resources:
       limits:
         cpu: "1"
         memory: "512Mi"
       requests:
         cpu: "0.5"
         memory: "256Mi"
     ```
   - Apply the updated deployment:
     ```bash
     kubectl apply -f <controller-deployment.yaml> -n kube-system
     ```

---

## General Observations from Recent Sources
- **AWS Load Balancer Controller Adoption**: Recent documentation emphasizes using the AWS Load Balancer Controller over the legacy Kubernetes Service Controller for better ALB management. Issues often arise when using deprecated controllers or incorrect annotations.[](https://docs.aws.amazon.com/eks/latest/best-practices/load-balancing.html)[](https://aws.github.io/aws-eks-best-practices/networking/loadbalancing/loadbalancing/)
- **Microservices Complexity**: ALB’s advanced routing (path-based or host-based) is widely used in microservices, but misconfigurations (e.g., incorrect path patterns or excessive rules) are common pain points.[](https://nidhiashtikar.medium.com/amazon-eks-leveraging-an-application-load-balancer-cf158caa4525)[](https://aws.amazon.com/blogs/containers/how-to-leverage-application-load-balancers-advanced-request-routing-to-route-application-traffic-across-multiple-amazon-eks-clusters/)
- **Fargate Integration**: Issues specific to Fargate (e.g., IP target mode) include ensuring proper subnet tagging and pod readiness gates to avoid registration delays.[](https://repost.aws/knowledge-center/eks-alb-ingress-controller-fargate)
- **Community Feedback**: X posts highlight ongoing challenges with gRPC and 503 errors, indicating these are persistent pain points in 2025.

## Recommendations
- **Automate with IaC**: Use Terraform or Helm to manage ALB and Ingress configurations to avoid manual errors.
- **Monitor Proactively**: Enable ALB access logs and CloudWatch metrics (`RequestCount`, `TargetResponseTime`, `HealthyHostCount`) to detect issues early.
- **Stay Updated**: Use the latest AWS Load Balancer Controller version (e.g., 2.11.0 or later as of 2025) to benefit from bug fixes and gRPC improvements.[](https://repost.aws/knowledge-center/eks-alb-ingress-controller-fargate)
- **Test Incrementally**: Deploy a sample application (e.g., the 2048 game from AWS docs) to validate ALB setup before production workloads.[](https://repost.aws/knowledge-center/eks-alb-ingress-controller-setup)

---