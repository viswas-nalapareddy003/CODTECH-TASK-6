# CODTECH-TASK-6
Step-by-Step Guide
1. Build a Web Application
For demonstration, let’s use a simple Node.js web application:

server.js

javascript
Copy code
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.send('Hello, GKE!');
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
package.json

json
Copy code
{
  "name": "gke-docker-app",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
2. Create a Dockerfile
The Dockerfile specifies how to build the Docker image:

Dockerfile

dockerfile
Copy code
# Use the Node.js image
FROM node:16

# Set the working directory
WORKDIR /usr/src/app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application
COPY . .

# Expose the port and run the application
EXPOSE 3000
CMD ["npm", "start"]
3. Build and Test the Docker Image
Build the Docker image and test it locally:

bash
Copy code
# Build the Docker image
docker build -t gke-docker-app .

# Run the Docker container
docker run -p 3000:3000 gke-docker-app
Visit http://localhost:3000 to confirm the app is running.

4. Push the Image to Google Container Registry (GCR)
Authenticate with Google Cloud:

bash
Copy code
gcloud auth login
gcloud config set project <your-gcp-project-id>
Tag and push the Docker image:

bash
Copy code
docker tag gke-docker-app gcr.io/<your-gcp-project-id>/gke-docker-app
docker push gcr.io/<your-gcp-project-id>/gke-docker-app
5. Set Up Google Kubernetes Engine (GKE)
Enable GKE in your project:

bash
Copy code
gcloud services enable container.googleapis.com
Create a GKE cluster:

bash
Copy code
gcloud container clusters create gke-cluster --num-nodes=3
Connect to the cluster:

bash
Copy code
gcloud container clusters get-credentials gke-cluster
6. Create Kubernetes Deployment and Service
Create YAML files for Kubernetes resources:

deployment.yaml

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gke-docker-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: gke-docker-app
  template:
    metadata:
      labels:
        app: gke-docker-app
    spec:
      containers:
      - name: gke-docker-app
        image: gcr.io/<your-gcp-project-id>/gke-docker-app
        ports:
        - containerPort: 3000
service.yaml

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: gke-docker-app-service
spec:
  type: LoadBalancer
  selector:
    app: gke-docker-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
7. Deploy to GKE
Apply the YAML configurations to deploy the application:

bash
Copy code
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
8. Access the Application
Run the following to get the external IP of the service:

bash
Copy code
kubectl get service gke-docker-app-service
Open the external IP in your browser to access the application.

9. Verify and Monitor
Use kubectl get pods to check the pod status.
Use kubectl logs <pod-name> to view application logs.
Use the GKE dashboard in the Google Cloud Console to monitor your cluster.

