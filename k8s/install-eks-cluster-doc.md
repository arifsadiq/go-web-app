# Prerequisites

1. Install kubectl

    Linux: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

    MacOS: https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/

    Windows: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/


2. Install eksctl
 
    Refer the link https://eksctl.io/installation/

3. Install AWS CLI

    Refer the link https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

# Configure AWS in your machine

```bash
aws configure
```

# Install an EKS cluster

```bash
eksctl create cluster --name <cluster-name> --region <region>
```

# Delete the cluster

```bash
eksctl delete cluster --name <cluster-name> --region <region>
```


