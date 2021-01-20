# Gerar Certificado para VAULT e CONSUL

### Baixar ferramentas
```sh
go get -u github.com/cloudflare/cfssl/cmd/cfssl
go get -u github.com/cloudflare/cfssl/cmd/cfssljson
```
Para funcionar o comandos a cima, precisa ter o PATH do GO em seu computador
```sh
mkdir $HOME/go
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```
### Dentro da pasta "certs" existe os arquivos:
- ca-config.json
- ca-csr.json
- consul-csr.json
- vault-csr.json

### Criar Autoridade Certificadora:

```sh
cfssl gencert -initca certs/config/ca-csr.json | cfssljson -bare certs/ca
```

### Criando private key e TLS certificado para o Consul:
```sh
cfssl gencert \
    -ca=certs/ca.pem \
    -ca-key=certs/ca-key.pem \
    -config=certs/config/ca-config.json \
    -profile=default \
    certs/config/consul-csr.json | cfssljson -bare certs/consul
```
### Criando private key e TLS certificado para o Vault
```sh
cfssl gencert \
    -ca=certs/ca.pem \
    -ca-key=certs/ca-key.pem \
    -config=certs/config/ca-config.json \
    -profile=default \
    certs/config/vault-csr.json | cfssljson -bare certs/vault
```

# CONFIGURAÇÃO CONSUL

### Para este passo, é necessário ter o client do consul na maquina, assim é possivel interagir com o mesmo após a instalação:

> Para instalar, utilize o link: https://www.consul.io/downloads

### Criando variavel para a instalação do consul
```sh
export GOSSIP_ENCRYPTION_KEY=$(consul keygen)
```
### Salvando os certificados dentro de uma secret no kubernetes
```sh
kubectl create secret generic consul \
  --from-literal="gossip-encryption-key=${GOSSIP_ENCRYPTION_KEY}" \
  --from-file=certs/ca.pem \
  --from-file=certs/consul.pem \
  --from-file=certs/consul-key.pem
```
>PS: Verifique se foi criado com sucesso
> ```sh
> kubectl describe secrets consul
> ```
> Dentro da Pasta Consul, existe os arquivo de configuração do SERVICE, INGRESS, STATEFULSET e CONFIGMAP
> para inserir no kubernetes, digite:
```sh
kubectl apply -f consul/
```
> PS: Verifique se foi criado com sucesso
```sh
kubectl get all; kubectl get ing; kubectl get secrets; kugectl get configmap
```
### Será criado o acesso até a plataforma do consul, pela porta 8500


## CONFIGURAÇÃO VAULT

### Crie a secret com o certificado para o vault
```sh
kubectl create secret generic vault \
    --from-file=certs/ca.pem \
    --from-file=certs/vault.pem \
    --from-file=certs/vault-key.pem
```
> PS: Verifique se foi criado
```sh
kubectl describe secrets vault
```
> Na mesma pasta, existe a estrutura do kubernetes para o vault, basta executar:
```sh
kubectl apply -f vault/
```
#V## erifique se toda a estrutuar foi criada
```sh
kubectl get all; kubectl get ing; kubectl get configmap; kubectl get secrets
```
## CRIANDO  DEPLOY VAULT e CONSUL
```sh
kubectl apply -f vault-consul-deployment.yaml
```

> se tudo foi criado, teste o acesso via navegador

> CONSUL:
> ip:8500
>VAUL:
>ip:8200
