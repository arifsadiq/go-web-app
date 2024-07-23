# Step-by-Step instructions to implement the go-web-app project

### Step 1: Clone the repository

```bash
git clone https://github.com/arifdevopstech/go-web-app.git
```

### Step 2: Remove the following files and folders

.github

gitops

Dockerfile

helmchart

ingress-controller

k8s

### Step 3: Create a multi-stage Docker file to reduce the image size and enhance security

                     FROM golang:1.22.5 as base      # specifies the base image

                     WORKDIR /app                   # set the working directory inside the container

                     COPY go.mod ./                 # copies the go.mod file to the dir of container

                     RUN go mod download            # executes commands in the container to install the dependencies

                    COPY . .                        # copies all the files from the current directory of host machine to the directory of container
                    
                    RUN go build -o main .          # executes build command

                    #############################################################################
                    ################ Reduce the image size using multi-stage builds. ############  
                    #############################################################################

                    FROM gcr.io/distroless/base            # specifies distroless image as base

                    COPY --from=base /app/main .           # copies the binary from the above stage         

                    COPY --from=base /app/static ./static  # copies the static files from the above stage

                    EXPOSE 8080                            # inform Docker that the container will listen on the specified network port at runtime

                    CMD [ "./main" ]                       # specifies the command to run when the container starts                

### Step 4: Build the docker image and run it locally and then push it to Dockerhub

```bash
docker build -t <docker-username/repo-name>:tag .

docker run -d -p 8080:8080 <docker-username/repo-name>:tag

docker push <docker-username/repo-name>:tag
```

### Step 5: Create an EKS cluster. As a prerequisites, install kubectl, eksctl and awscli

```bash
aws configure

eksctl create cluster --name <cluster-name> --region <region>
```

### Step 6: Create an Ingress Controller. In this project we are using Nginx Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
```

### Step 7: Create Helm chart

Create a folder 'helm' in the root directory and navigate to it

```bash
helm create go-web-app-chart
```
This will create the helm chart directory along with common files and folders. Delete the "chart" directory and the entire contents from the templates directory

Navigate to the templates directory and create the following Kubernetes manifests files. You can either refer to my github repo or use k8s documentation to write the manifests listed below

    1. deployment.yaml
    2. service.yaml
    3. ingress.yaml

In the values.yaml file we will parameterize the repository and tag. You can refer to my github repo to understand more.

To install the manifests files using helm

```bash
helm install go-web-app ./go-web-app-chart
```

### Step 8: Add the domain go-web-app.local in hosts file

In the ingress.yaml file, since the host is defined as go-web-app.local, we need to add this in our hosts file so the application will run on go-web-app.local

To do this, we need to find the IP Address of the ingress. To get that, execute the commands below:

```bash
kubectl get ingress
```
```bash
nslookup <xxxxxxxxxxxxxxxxxx.elb.region.amazonaws.com>
```
You can find the IP address from Non-authorative answer. Once you get the IP, open your hosts file and add the following and save it.

```bash
<IP-Address>    go-web-app.local
```

### Step 9: Create CI using GitHub Actions

> Create the directory .github/workflows in root folder and create an yaml file. In that yaml file we are writing the steps to implement the CI part.
  Steps involved in CI
> 
    1. Build and Unit Test
    
    2. Static code Analysis
    
    3. Docker build and push
    
    4. Update the image tag at helm chart

> To create the secrets such as DOCKERHUB_USERNAME and DOCKERHUB_TOKEN, go to repository --> settings --> Secrets and variables --> Actions --> New repository secret

> The one which I used as TOKEN_GITHUB is the github token. You can create the github token and add it as New repository secret

> To generate the DOCKERHUB_TOKEN, login to your dockerhub --> Account settings --> Personal access tokens --> generate new token --> Provide a description and scope as Read & Write

Run the CI by commiting the changes and you can track this from Actions tab in your repository

### Step 9: Install ArgoCD

To install the ArgoCD using manifests

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Edit the service 'argocd-server' and change the type from ClusterIP to LoadBalancer

```bash
kubectl edit service/argocd-server -n argocd
```

To get the Load balancer IP

```bash
kubectl get svc argocd-server -n argocd
```
Using the external IP you can access the ArgoCD on browser after few minutes

Username is "admin" and to retrieve the password follow the below commands:

> List the secrets

```bash
kubectl get secret -n argocd
```

> To get the password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd
```

Copy the password and it is encoded with base64. We have to decode it.

> To decode the password

```bash
echo <password> | base64 --decode
```
replace password with your encoded password


### Step 10: Configure ArgoCD

Once you get into ArgoCD

1. Go to NEW APP
2. Application Name :  go-web-app
3. Project Name: default
4. SYNC POLICY: Automatic
5. Check Self heal box
6. Repository URL : https://github.com/arifdevopstech/go-web-app.git [copy yours]
7. Revision: HEAD
8. Path: ArgoCD will identify the path. select that one
9. Cluster URL: https://kubernetes.default.svc
10. Namespace: default
11. VALUES FILES: values.yaml

Once done, click on CREATE





