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

### 7 - Install ChaosMesh

- Adicione o repositório necessário no helm

```console
helm repo add chaos-mesh https://charts.chaos-mesh.org
```

- Crie o namespace para instalação (Opcional)

```console
kubectl create ns chaos-testing
```

- Instale via helm

```console
helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-testing --set chaosDaemon.runtime=containerd --set chaosDaemon.socketPath=/run/containerd/containerd.sock
```

- Espere todos os pods estarem em running e liste todos os services

```console
kubectl -n chaos-testing get svc
```

- Para acessar o dashboard, execute um portfowarding no serviço do dashboard

### 8 - Cleanup. 

- Para deletar o cluster, basta seguir a documentação abaixo.

https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html
