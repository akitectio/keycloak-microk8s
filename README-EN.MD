Below are the steps to deploy Keycloak on Kubernetes:

**Step 1: Install Microk8s**

Refer to the article: [Microk8s](https://ducduy.dev/en/categories/microk8s) to install Microk8s.

**Step 2: Create the `keycloak-service.yaml` file**

Create a service file to expose the Keycloak application. Keycloak operates by default on port 8080. For the latest version, you need to create a `headless` service, which enables High Availability (HA) Keycloak on Kubernetes.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: keycloak
  labels:
    app: keycloak
spec:
  ports:
    - name: http
      port: 8080
      targetPort: 8080
  selector:
    app: keycloak
  type: ClusterIP
  clusterIP: None
```

After creating the service YAML, apply the deployment with the following command:

```bash
microk8s kubectl apply -f service.yaml -n keycloak
```

To check if the service has been created, run the following command:

```bash
microk8s kubectl get svc -n keycloak
```

**Step 3: Create the `keycloak-deployment.yaml` file**

Create a deployment file for Keycloak with the following configuration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
  labels:
    app: keycloak
spec:
  replicas: 2
  selector:
    matchLabels:
      app: keycloak
  template:
    metadata:
      labels:
        app: keycloak
    spec:
      containers:
        - name: keycloak
          image: quay.io/keycloak/keycloak:22.0.4
          args: ["start", "--cache-stack=kubernetes"]
          env:
            # Add environment variables here
          ports:
            - name: jgroups
              containerPort: 7600
            - name: https
              containerPort: 8443
            - name: http
              containerPort: 8080
          readinessProbe:
            httpGet:
              scheme: HTTP
              path: /health/ready
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 1
```

If you have multiple replicas, adjust the environment variables for `CACHE_OWNERS_COUNT` and `CACHE_OWNERS_AUTH_SESSIONS_COUNT` accordingly.

Apply the deployment using the following command:

```bash
microk8s kubectl apply -f keycloak-deployment.yaml -n keycloak
```

To check if the deployment has been created, run the following command:

```bash
microk8s kubectl get deployments -n keycloak
```

**Step 4: Create the `keycloak-ingress.yaml` file**

Create an Ingress resource to route traffic to Keycloak:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: keycloak-ingress
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: 2500m
    nginx.ingress.kubernetes.io/proxy-buffer-size: 12k
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  ingressClassName: nginx
  rules:
    - host: keycloak.localhost
      http:
        paths:
          - backend:
              service:
                name: keycloak
                port:
                  number: 8080
            path: /(.*)
            pathType: Prefix
```

Apply the Ingress resource using the following command:

```bash
microk8s kubectl apply -f keycloak-ingress.yaml -n keycloak
```

To check if the Ingress has been created, run the following command:

```bash
microk8s kubectl get ingress -n keycloak
```

After deploying the service, deployment, and Ingress, you can check the status of the pods by running the following command:

```bash
microk8s kubectl get pods -n keycloak
```

This will help you verify if the Keycloak components are running.
