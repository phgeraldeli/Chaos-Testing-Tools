# Gremlin



### Requisitos:

- Conta na AWS

### 1 - Configuração do awscli

- Verifique a instalação do awscli

```console
aws --version
```

- Configure a awscli com suas credenciais.
```
aws configure
```

### 2 - Baixe o eksclt

https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html

### 3 - Instalar o kubectl

- Instale o kubectl para acessar o cluster.

https://kubernetes.io/docs/tasks/tools/

### 4 - Crie o cluster eks.

- O comando a baixo vai criar um cluster de EKS com 4 subnets, 2 zonas, 2 máquinas m5.large

```console
eksctl create cluster
```

Se for de sua preferencia é possivel passar um arquivo de configuração para o eksctl da seguinte forma:

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: basic-cluster
  region: us-east-1

nodeGroups:
  - name: ng-1
    instanceType: m5.large
    desiredCapacity: 10
    volumeSize: 80
    ssh:
      allow: true # will use ~/.ssh/id_rsa.pub as the default ssh key
  - name: ng-2
    instanceType: m5.xlarge
    desiredCapacity: 2
    volumeSize: 100
    ssh:
      publicKeyPath: ~/.ssh/ec2_id_rsa.pub
```

```console
eksctl create cluster -f config.yaml
```


### 5 - Configure o acesso ao cluster

```console
sudo aws eks --region us-east-1  update-kubeconfig --name cluster_name
```

- Teste se o cluster está acessivel.

```console
kubectl cluster-info
```

### 6 - Install Helm

https://helm.sh/docs/intro/install/

### 7 - Instalação Gremlin

- Acesse o gremlin e pegue o teamID e o secretID do seu time.

![img2](https://user-images.githubusercontent.com/22543429/129482711-f0807747-098c-41e8-87a8-0dca41a1a64d.png)
![img3](https://user-images.githubusercontent.com/22543429/129482720-936dd809-fa84-4d30-8fe8-101c775d8496.png)
![img4](https://user-images.githubusercontent.com/22543429/129482724-227a2f93-a759-4b86-9ae2-141b3c547bd1.png)


- Crie um namespace para o gremlin

```console
kubectl create namespace gremlin
```

- Use o helm para instalação do gremlin.

```console
helm install gremlin gremlin/gremlin \
    --namespace gremlin \
    --set gremlin.secret.managed=true \
    --set gremlin.secret.type=secret \
    --set gremlin.secret.teamID=$GREMLIN_TEAM_ID \
    --set gremlin.secret.clusterID=$GREMLIN_CLUSTER_ID \
    --set gremlin.secret.teamSecret=$GREMLIN_TEAM_SECRET
```

- Rode o comando abaixo e espere todos os pods subirem

```console
kubectl -n gremlin get pods -w
```

A instalação está finalizada. 

Segue a documentação para excluir o cluster: https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html
