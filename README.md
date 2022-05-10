# igti-edc-desafio_final
Desafio final do bootcamp IGTI - Engenheiro de Dados Cloud 2022A

### 1. Configuração do ambiente

**Instalação do cliente AWS**
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
aws configure
```
**Instalação do kubectl**
```
curl -LO https://dl.k8s.io/release/v1.23.6/bin/linux/amd64/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version
```
**Instalação do eksctl**
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### 2. Criação do cluster Kubernetes na AWS

```
eksctl create cluster --name=k8sigtidesafiofinal --managed --instance-types=m5.large --spot --nodes-min=2 --nodes-max=4 --region=us-east-1 --alb-ingress-access --node-private-networking --full-ecr-access --nodegroup-name=ng-k8sigtidesafiofinal
```

### 3. Instalação e configuração da aplicação Spark Operator

**Criação da imagem no [Docker Hub](https://hub.docker.com)**
```
cd kubernetes/spark
docker build -t {dockerhub_account}/spark-operator:v1.0.0-aws .
docker login -u {dockerhub_account} -p {password}
docker push {dockerhub_account}/spark-operator:v1.0.0-aws
```
- OBS: Há uma imagem já disponível no endereço abaixo:
`neylsoncrepalde/spark-operator:v3.0.0-aws`

**Deploy do Spark Operator no Kubernetes**

**- Criação do namespace**
Mudar para spark-operator???
`kubectl create namespace desafio_final`

**- Criação da service account**
video aula 3.2. Spark 14:30min

video 3.2 spark 14:30min
`kubectl create serviceaccount spark -n desafio_final`

**- Criação da rolebind**
`kubectl create clusterrolebinding spark-role-binding -clusterrole=edit --serviceaccount=desafio_final:spark -n desafio_final`

**- Deploy do Helm Operator do Spark**
`helm repo add spark-operator https://googlecloudplatform.github.io/spark-on-k8s-operator`
`helm repo update`
`helm install spark spark-operator/spark-operator -n desafio_final`

**- Verificar se o spark operator está executando**
`kubectl get pods -n desafio_final`

video em 21min
a partir daqui ver se o complemento está na aula interativa do desafio final.

Inserir o código spark-operator-processing-job-batch.py automaticamente no storage???

Configuração do yaml do spark
 - alterar namespsame de processing para desafio_final (OK)
 - alterar destinos do S3
 - alterar nome do job para ser automático (datetime.now())

Criação das credenciais da AWS nas variáveis de ambiente 
`kubectl create secret generic aws-credentials --from-literal=aws_access_key_id={keyid} --from-literal=aws_secret_access_key={senha} -n desafio_final`

**- Deploy da Aplicação Spark** (EXECUÇÂO do JOB)

Apply no yaml do spark application
`kubectl apply -f spark-back-operator-k8s-v1beta2.yaml -n desafio_final`

### 4. Criar Crawlers na AWS
  1. edsup2019_crawler_aluno
  2. edsup2019_crawler_curso
  3. edsup2019_crawler_docente

### 5. Instalação e configuração da aplicação Airflow

Criação das credenciais da AWS nas variáveis de ambiente 
`kubectl create secret generic aws-credentials --from-literal=aws_access_key_id={keyid} --from-literal=aws_secret_access_key={senha} -n airflow`

- Deploy da rolebind
`kubectl apply -f kubernetes/airflow/rolebinding_for_airflow.yaml -n airflow`

 Alterar destino S3 do log no custom_values.yaml
 Alterar dados do defaultUser do webserver
 Apontar o gitSync para o meu repositório


- Deploy do Airflow
`helm install airflow apache-airflow/airflow -f kubernetes/airflow/custom_values -n airflow --debug`

- Pegar o endpoint para acesso externo ao Airflow
`kubectl get pods,svc -n airflow`
Pegar o endereço do LoadBalancer + :8080

- Trocar password do Airflow
- cadastrar as variaveis de ambiente:
  1. aws_access_key_id
  2. aws_secret_access_key

- Adicionar as connections:
  1. my_aws
    Type: Amazon Web Services
    Login : key id
    Password: secret da aws
  2. kubernetes_default
    Type: Kubernetes Cluster Connection
    Marcar a opção In cluster configuration
- ativar a DAG desafio_final_btc_edsup_2019

Continuação do video 46min



### . Remoção do cluster Kubernetes na AWS

antes:
apagar namespace?
apagar clusterrolebinding spark-role-binding
apagar serviceaccount spark
apagar helm operator do spark
apagar sparkapplication spark-back-operator-k8s-v1beta2.yaml
```
eksctl delete cluster --region us-east-1 --name k8sigtidesafiofinal
```