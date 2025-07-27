Deploying a Minimal Next.js App on Kubernetes with NGINX Ingress Controller
1. Creating a Minimal Next.js App
Start with a basic Next.js app structure:

* page.tsx renders your main page and image.
* layout.tsx sets up the layout and fonts.
* globals.css contains global styles.
This keeps your app simple and focused for deployment.

2. Building the Docker Image
Create a Dockerfile:
```Dockerfile
# Use official Node.js image as base
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

EXPOSE 3000
CMD ["npm", "start"]
```
Build and push your image to a registry accessible by your Kubernetes cluster.

3. Kubernetes Deployment and Service for the App
Create a deployment and service YAML (app-wear-app.service.yml):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-wear-service
  labels:
    app: app-wear
spec:
  type: NodePort
  selector:
    app: app-wear
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
      nodePort: 31000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-wear-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app-wear
  template:
    metadata:
      labels:
        app: app-wear
    spec:
      containers:
      - name: app-wear-app
        image: <your-registry>/app-wear-app:1.0.0
        ports:
        - containerPort: 3000
```
Explanation:

Deployment manages your app pods.
Service exposes your app internally and via NodePort for testing.
4. NGINX Ingress Controller and Related Kubernetes Objects
a. ServiceAccount
Grants the controller permissions to interact with the cluster.
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: default
```
b. ClusterRole & ClusterRoleBinding
Defines and binds permissions for the controller.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
rules:
  - apiGroups: [""]
    resources: ["configmaps", "endpoints", "nodes", "pods", "secrets", "services", "events"]
    verbs: ["get", "list", "watch", "create", "patch"]
  - apiGroups: ["extensions", "networking.k8s.io"]
    resources: ["ingresses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["get", "list", "watch", "create", "update"]
  - apiGroups: ["discovery.k8s.io"]
    resources: ["endpointslices"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: default
```
c. ConfigMap
Optional, for NGINX configuration.
```yaml
kind: ConfigMap
apiVersion: v1
metadata: 
  name: nginx-configuration
```
d. Deployment and Service
Deploys the controller and exposes it via NodePort.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: k8s.gcr.io/ingress-nginx/controller:v1.10.1
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
          ports:
            - containerPort: 80
              name: http
            - containerPort: 443
              name: https
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      name: http
    - protocol: TCP
      port: 443
      targetPort: 443
      name: https
  selector:
    name: nginx-ingress
```
Explanation:

ServiceAccount, ClusterRole, ClusterRoleBinding: Securely grant the controller access to required resources.
Deployment: Runs the controller pod.
Service: Exposes the controller for external access.
5. Ingress Resource (app-wear.ingress.yaml)
Defines how external traffic is routed to your app.
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-app-wear
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  ingressClassName: nginx
  rules:
    - host: appwear.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-wear-service
                port:
                  number: 80
```

Explanation:

Ingress maps requests for appwear.local to your app-wear service.
Annotations and ingressClassName ensure NGINX processes this resource.
6. Modifying the Hosts File
Add this line to your hosts file (hosts):
```hosts
127.0.0.1 appwear.local
```

Explanation:
This lets your browser resolve appwear.local to your local machine, which is required for Ingress routing.

7. Accessing the App via Ingress Controller
If using Minikube with Docker driver, start the tunnel:
```shell
minikube service nginx-ingress
```
Use the provided 127.0.0.1:<port> URL.

* In your browser, visit:
```
http://appwear.local:<tunnel-port>/
```
Or use curl:
```
curl -H "Host: appwear.local" http://127.0.0.1:<tunnel-port>/
```
8. Troubleshooting Tips
Pod not starting: Check ServiceAccount and RBAC permissions.
404 from NGINX: Host header mismatch or Ingress not applied.
Cannot connect: Use Minikube tunnel with Docker driver; direct NodePort access may not work.
Ingress not routing: Ensure ingressClassName and annotation are set.
Check logs:
```shell
kubectl logs -l name=nginx-ingress
kubectl describe ingress ingress-app-wear
kubectl get events
```
Conclusion
This workflow demonstrates how to containerize a Next.js app, deploy it to Kubernetes, and expose it securely and flexibly using NGINX Ingress Controller.
Each Kubernetes object plays a role in security, routing, and scalability.
Troubleshooting is essential—always check logs, events, and network settings.

