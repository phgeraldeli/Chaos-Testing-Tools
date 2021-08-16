# ChaosMesh

### 1 - Configuração do awscli

- Verifique a instalação do awscli

```console
aws --version
```

- Configure a awscli com suas credenciais.
```console
aws configure
```

### 2 - Baixe o eksclt

https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html

### 3 - Instalar o kubectl

- Instale o kubectl para acessar o cluster.

https://kubernetes.io/docs/tasks/tools/

### 4 - Instalar Helm

https://helm.sh/docs/intro/install/

### 5 - Suba o cluster EKS

```console
eksctl create cluster --name=eks-prod
```

### 6 - Configure o acesso ao cluster

```console
aws eks update-kubeconfig --name eks-prod
```

- Verifique o acesso ao cluster

```console
kubectl cluster-info
```

### 7 - 