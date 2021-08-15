# Chaos Monkey

# Requisitos 

- Um banco mysql
- Conta no dockerhub
- Go

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

### 4 - Install halyard

```console
curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/debian/InstallHalyard.sh
sudo bash InstallHalyard.sh
sudo update-halyard
hal -v
```

### 5 - Suba o cluster EKS

```console
eksctl create cluster --name=eks-prod
```

### 6 - Configure o acesso ao cluster

```console
aws eks update-kubeconfig --name eks-prod --region us-east-1 --alias eks-prod
```

### 7 - Crie e configure o docker registry

```console
hal config provider docker-registry enable 

hal config provider docker-registry account add ibuchh-docker \ 
 --address index.docker.io --username ibuchh --password
 ```
 
### 8 - Configure a conta do github

```console
hal config artifact github enable

hal config artifact github account add spinnaker-github --username ibuchh \
  --password --token
 ```
 
### 9 - Configure as contas kubernetes

```console
hal config provider kubernetes enable

kubectl config use-context eks-prod

CONTEXT=$(kubectl config current-context)
```
 
### 10 - Crie service account para o EKS cluster

```console
kubectl apply --context $CONTEXT \
    -f https://spinnaker.io/downloads/kubernetes/service-account.yml
    
TOKEN=$(kubectl get secret --context $CONTEXT \
   $(kubectl get serviceaccount spinnaker-service-account \
       --context $CONTEXT \
       -n spinnaker \
       -o jsonpath='{.secrets[0].name}') \
   -n spinnaker \
   -o jsonpath='{.data.token}' | base64 --decode)

kubectl config set-credentials ${CONTEXT}-token-user --token $TOKEN

kubectl config set-context $CONTEXT --user ${CONTEXT}-token-user

hal config provider kubernetes account add eks-prod --provider-version v2 \
 --docker-registries ibuchh-docker --context $CONTEXT
 
hal config features edit --artifacts true
```


### 11 - Instalação Spinnaker

- Configure o spinnaker para instalação no kubernetes

```console
hal config deploy edit --type distributed --account-name eks-spinnaker
```

- Configure o halyard para armazenar as configs no s3

```console
export YOUR_ACCESS_KEY_ID=<access-key>

hal config storage s3 edit --access-key-id $YOUR_ACCESS_KEY_ID \
 --secret-access-key --region us-east-2
 
 hal config storage edit --type s3
 ```
 
 - Escolha a versão do spinnaker a ser instalado
 
 ```console
 hal version list
 
 export VERSION=1.15.0

hal config version edit --version $VERSION

hal deploy apply
```

- Acompanhe o deploy com os comandos
```console
kubectl -n spinnaker get svc
```

```console
kubectl -n spinnaker get pods -w
```

- Para expor o Spinnaker através de um Load Balancer faça:


```console
export NAMESPACE=spinnaker

kubectl -n ${NAMESPACE} expose service spin-gate --type LoadBalancer \
  --port 80 --target-port 8084 --name spin-gate-public 

kubectl -n ${NAMESPACE} expose service spin-deck --type LoadBalancer \
  --port 80 --target-port 9000 --name spin-deck-public  

export API_URL=$(kubectl -n $NAMESPACE get svc spin-gate-public \
 -o jsonpath='{.status.loadBalancer.ingress[0].hostname}') 

export UI_URL=$(kubectl -n $NAMESPACE get svc spin-deck-public  \ 
 -o jsonpath='{.status.loadBalancer.ingress[0].hostname}') 

hal config security api edit --override-base-url http://${API_URL} 

hal config security ui edit --override-base-url http://${UI_URL}

hal deploy apply
```



### 12 - Instalação Chaos Monkey

Com a instalação do spinnaker dentro do EKS, basta seguir o passo a passo na documentação oficial e rodar os testes.

https://netflix.github.io/chaosmonkey/How-to-deploy/

### 13 - Configurando os testes

- Para configurar os testes, podemos rodar localmente ou configurar via spinnaker.

https://netflix.github.io/chaosmonkey/Configuring-behavior-via-Spinnaker/

Após a configuração podemos rodar o comando abaixo e verificar a configuração feita.

```console
chaosmonkey config spinnaker
```

Segue a documentação para excluir o cluster: https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html
