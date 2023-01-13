## Configurando MongoDB no Kubernetes

Antes de subir a aplicação do backend NestJS no Kubernetes, vamos precisar deixar o banco de dados configurado. Não é recomendável utilizar banco de dados no Kubernetes, mas aqui ele será utilizado para facilitar nosso estudo. Banco de dados é algo tão sensível, com tantas configurações e necessidade de performance, que não faz sentido usar um banco de dados com Kubernetes no mundo real.

Para facilitar ainda mais a configuração do banco de dados, iremos utilizar uma ferramenta chamada **Helm Charts**, que é uma espécie de pacote de manifestos do Kubernetes para fazer a configuração de diversos tipos de aplicações. Para o MongoDB, utilizaremos o [Bitnami MongoDB](https://github.com/bitnami/charts/tree/main/bitnami/mongodb), que instala o MongoDB no Kubernetes sem que você precise fazer configurações como Config Maps, Deployments, etc.

Vamos às configurações:

2. Adicionando o Helm no projeto:
`helm repo add helm-mongodb https://charts.bitnami.com/bitnami`

3. Adicionando o MongoDB no projeto através do Helm:
`helm install mongodb helm-mongodb/mongodb --set=auth.rootPassword="root",auth.database="nest",auth.username="root"`
*Obs: Note que estamos passando algumas configurações adicionais para configurar nome do banco de dados, usuário de acesso e senha*

4. Vamos precisar entrar no banco de dados para criarmos os 3 trajetos com os quais estamos trabalhando. Mas antes de entrarmos no MongoDB, vamos guardar a senha dele antes:
`export MONGODB_ROOT_PASSWORD=$(kubectl get secret --namespace default mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 -d)`
*Obs: Aqui pegamos a senha do **secret** que foi gerado.*

5. Entrando no bash do MongoDB:
`kubectl run --namespace default mongodb-client --rm --tty -i --restart='Never' --env="MONGODB_ROOT_PASSWORD=$MONGODB_ROOT_PASSWORD" --image docker.io/bitnami/mongodb:6.0.2-debian-11-r1 --command -- bash`

6. Conectando ao MongoDB:
`mongosh admin --host "mongodb" --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD`

7. Entrando no nosso banco de dados *nest*:
`use nest`

8. Inserindo os dados:
```
db.routes.insertMany([
    {
        _id: '1',
        title: 'Primeiro',
        startPosition: { lat: -15.82594, lng: -47.92923 },
        endPosition: { lat: -15.82942, lng: -47.92765 },
    },
    {
        _id: '2',
        title: 'Segundo',
        startPosition: { lat: -15.82449, lng: -47.92756 },
        endPosition: { lat: -15.8276, lng: -47.92621 },
    },
    {
        _id: '3',
        title: 'Terceiro',
        startPosition: { lat: -15.82331, lng: -47.92588 },
        endPosition: { lat: -15.82758, lng: -47.92532 },
    }
]);
```

*Obs: Você pode verificar se o MongoDB está funcionando corretamente executando os seguintes comandos*
```
kubectl get pods

kubectl get services
```