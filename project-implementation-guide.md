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

### Step 4: Build the docker image and test it locally


