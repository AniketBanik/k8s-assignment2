# Kubernetes Assignment 2 - Deployment Report

## Author: Aniket Banik

**GitHub Repository:** [k8s-assignment2](https://github.com/AniketBanik/k8s-assignment2)

---

## **1. Introduction**

This report provides an overview of the Kubernetes manifests used in this assignment, challenges faced during deployment, solutions implemented, and explanations regarding service types and ReplicaSets.

---

## **2. Kubernetes Manifest Files**

This repository contains the following Kubernetes manifests:

- `` - Defines a MySQL Pod for database services.
- `` - Describes the deployment for a web application, ensuring high availability using ReplicaSets.
- `` - Configures a NodePort service to expose the web application externally.

---

## **3. Challenges Faced & Solutions**

### **Challenge 1: ImagePullBackOff Issue**

- **Issue:** The web application pods were stuck in `ImagePullBackOff` due to incorrect authentication with AWS ECR.
- **Solution:**
  - Logged into AWS ECR manually using:
    ```bash
    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ECR_URL>
    ```
  - Updated `imagePullSecrets` in `web-deployment.yaml`.
  - Restarted the deployment using:
    ```bash
    kubectl rollout restart deployment web-deployment
    ```

### **Challenge 2: NodePort Service Not Accessible Externally**

- **Issue:** Unable to access the web application via `curl http://<EC2_PUBLIC_IP>:30000`.
- **Solution:**
  - Modified **Security Group Rules** in AWS to allow TCP traffic on port `30000`.
  - Verified service configuration:
    ```bash
    kubectl describe svc web-service
    ```
  - Restarted `kube-proxy` to apply changes:
    ```bash
    kubectl rollout restart daemonset/kube-proxy -n kube-system
    ```

### **Challenge 3: Pods Running But No Response from Web App**

- **Issue:** The web application was running inside pods but did not respond via the NodePort service.
- **Solution:**
  - Verified container logs using:
    ```bash
    kubectl logs -l app=web --tail=50
    ```
  - Checked pod-to-service connectivity by running a temporary debug pod:
    ```bash
    kubectl run debug --rm -it --image=busybox -- /bin/sh
    wget -O- http://10.96.110.248:80
    ```
  - Confirmed the app was listening on the correct port (`8080`).

---

## **4. Answers to Assignment Questions**

### **Q1: What is the IP of the Kubernetes API Server?**

- The Kubernetes API Server IP can be found using:
  ```bash
  kubectl cluster-info
  ```
  Example Output:
  ```
  Kubernetes master is running at https://kind-control-plane:6443
  ```

### **Q2: How does a ReplicaSet work?**

- A **ReplicaSet** ensures that a specified number of pod replicas are running at all times.
- If a pod fails or is deleted, the ReplicaSet automatically creates a new one.
- This provides **fault tolerance** and **scalability**.

### **Q3: Why is a NodePort service used for the web application?**

- **NodePort** exposes the service on each node's IP at a static port (`30000` in this case).
- This allows external access without requiring a load balancer.
- Example of service definition in `web-service.yaml`:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: web-service
  spec:
    type: NodePort
    ports:
      - port: 80
        targetPort: 8080
        nodePort: 30000
    selector:
      app: web
  ```

---

## **5. Summary of Deployment Steps**

1. **Setup Kubernetes Cluster**
   ```bash
   kubectl get nodes
   ```
2. **Deploy MySQL Pod**
   ```bash
   kubectl apply -f mysql-pod.yaml
   ```
3. **Deploy Web Application**
   ```bash
   kubectl apply -f web-deployment.yaml
   ```
4. **Expose Web Application using NodePort**
   ```bash
   kubectl apply -f web-service.yaml
   ```
5. **Verify Everything is Running**
   ```bash
   kubectl get all
   ```
6. **Test Application Accessibility**
   ```bash
   curl http://<EC2_PUBLIC_IP>:30000
   ```

---

## **6. Conclusion**

- Successfully deployed a **Kubernetes web application** with MySQL backend.
- Addressed deployment challenges, ensuring **external access** via `NodePort`.
- The setup is scalable with ReplicaSets ensuring availability.
- All configurations are stored in this GitHub repository for future reference.

---

## **7. Video Demonstration**

> 📌 A recording of the deployment and update process will be submitted as per assignment requirements.

*Submission Checklist:**

✔ Kubernetes manifests pushed to [GitHub](https://github.com/AniketBanik/k8s-assignment2)\
✔ README.md report added\
✔ Video demonstration recorded & uploaded

---

