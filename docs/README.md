## Informações e instruções importantes

1. Para que o Docker possa resolver o **host.docker.internal**, é necessário incluir o endereço **127.17.0.1** no **/etc/hosts** da sua máquina.

    *Obs: No Windows, você vai encontrar este arquivo no seguinte diretório **C:\Windows\System32\drivers\etc***

    A seguinte linha deve ser inserida ao final do arquivo:

    `172.17.0.1 host.docker.internal`

2. Ainda pode ser que você venha a ter um problema relacionado ao consumo de memória pelo Docker, caso esteja utilizando WSL (Windows Subsystem for Linux). Neste caso, execute o seguinte comando dentro do seu Linux:

    `sudo sysctl -w vm.max_map_count=262144`

    *Você pode aumentar ou diminuir o valor, caso achar necessário.*

3. Em sistemas Linux, pode ser que você não consiga subir o Kibana e, ao tentar acessá-lo em *localhost:5601*, obtenha a seguinte mensagem de erro: ***Kibana server is not ready yet***. Isto acontece porque o diretório *es01* não tem as permissões corretas para que o Elastic Search que está rodando no container possa ler e escrever nele. Para corrigir este problema, basta navegar até onde se encontra o diretório *es01* e executar o seguinte comando:

    `sudo chown -R 1000:root es01`

4. Caso você tenha feito o deploy do projeto em um cluster Kubernetes, se você tentar acessar a aplicação por meio do IP entregue pelo **service** do Kubernetes, vai notar que o mapa não aparece e um erro **GeolocationPositionError** é exibido no console do navegador com uma ***message: "Only secure origins are allowed (see: https://goo.gl/Y0ZkNV)".*** Isso acontece porque não temos nada de TLS configurado, e, por padrão, a API do Google Maps só aceita ser acessada por meio de uma **origem https**. Para testes, podemos contornar isso da seguinte maneira:

    * Acesse no seu navegador **chome://flags**;
    * Encontre a opção **Insecure origins treated as secure**, habilite-a e insira o endereço de IP que você obteve pelo service do Kubernetes.
    *Obs: Você pode consultar o endereço de IP do seu service consultando: kubectl get services*;
    * Reinicie o navegador e tente acessar novamente.

## Executando aplicação localmente

Para executar aplicação no ambiente local, basta executar `docker-compose up` no diretório de cada aplicação.

A ordem é importante, portanto vamos subir:
1. Kafka
2. Simulator Go
3. Backend NestJS
4. Frontend ReactJS

*Obs: É importante lembrar que, para conectar ao Apache Kafka executando em container local, é necessário comentar as configurações de **ssl** e **sasl** nos arquivos:*
* *simulator/infra/kafka/consumer.go*
* *simulator/infra/kafka/producer.go*
* *nest-api/src/main.ts*
* *nest-api/src/routes/routes.module.ts*


## Preparativos para o deploy

Antes de prosseguir para o deploy, você deve garantir que seu **kubectl** já está conectado ao seu cluster Kubernetes. Neste caso, eu conectei num cluster da Google Cloud usando o *gcloud cli*.

Além disso, do modo como o projeto foi desenvolvido, você vai precisar criar um cluster Kafka no site da [Confluent](https://confluent.cloud/), criar nele os tópicos **route.new-direction** e **route.new-position**, configurar um client (pode ser um client Go), e, deste client, pegar o **username**, o **password** e o **bootstrap-servers** para usar nos **Config Maps** e nos **.env**.

Ainda, antes de prosseguir, você vai precisar subir as imagens do Simulator Go, do Backend NestJS e do Frontend ReactJS para **Docker Hub**, de modo que o Kubernetes possa acessá-las para subir suas aplicações. Para isto, acesse a pasta de cada uma dessas aplicações e execute:

```
sudo docker build -t your_docker_username/your_image_name -f Dockerfile.prod .

sudo docker push your_docker_username/your_image_name
```
*Obs: No caso do Frontend ReactJS, não se esqueça de atualizar o **.env** antes de subir para o Docker Hub, visto que os dados dele são passados para a aplicação no momento do build.*

Por fim, agora que você já tem as imagens no Docker Hub, atualize os arquivos **deploy.yaml** que estão na pasta **k8s** com as imagens que você subiu.

## Fazendo deploy com o Kubernetes

1. Simulator Go
    * Execute os seguintes comandos:
        ```
        kubectl apply -f k8s/simulator/configmap.yaml

        kubectl apply -f k8s/simulator/deploy.yaml
        ```
2. MongoDB
    * Siga as instruções detalhadas no README da pasta **mongodb**

3. Backend NestJS
    * A configuração inicial do backend é muito parecida com a do Simulator Go, o que muda são os nomes de algumas variáveis de ambiente no ConfigMap. Execute então os seguintes comandos:
        ```
        kubectl apply -f k8s/backend/configmap.yaml

        kubectl apply -f k8s/backend/deploy.yaml
        ```
    * Para que seja possível acessarmos este pod externamente, precisamos gerar um IP de modo que todo mundo que acesse o IP seja direcionado para este pod. Por isso utilizarmos o **service** do tipo **LoadBalancer**. Para isto, execute:
        ```
        kubectl apply -f k8s/backend/service.yaml
        ```
        *Obs: No Kubernetes, à medida que você começa a subir muitos services no seu dia-a-dia, é recomendável trocar o LoadBalancer por um Ingress. Este trabalha com apenas um IP, onde você manda todo o tráfego e ele distribui os serviços.*

        *No mundo real, se você tiver muitos services expostos para a rede, você não vai querer ficar gastando dinheiro com IP e coisas do tipo, portanto, o Ingress faz mais sentido.*

4. Frontend ReactJS
    * No frontend, também utilizamos variáveis de ambiente, porém, elas elas serão geradas somente na hora de fazer o build do projeto, pois teremos um projeto estático. Para resolver este problema, execute `kubectl get services`, pegue o IP exibido e coloque-o no **.env**
    * Agora, para subir as configurações, você pode executar:
        ```
        kubectl apply -f k8s/frontend/
        ```
        *Obs: Executando o comando desta forma, você aplica todas as configurações que estão na pasta **frontend**.* 
