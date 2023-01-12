## Informações e instruções importantes

1. Para que o Docker possa resolver o **host.docker.internal**, é necessário incluir o endereço **127.17.0.1** no **/etc/hosts** da sua máquina.

    *Obs: No Windows, você vai encontrar este arquivo no seguinte diretório **C:\Windows\System32\drivers\etc***

    A seguinte linha deve ser inserida ao final do arquivo:

    `172.17.0.1 host.docker.internal`

2. Ainda pode ser que você venha a ter um problema relacionado ao consumo de memória pelo Docker, caso esteja utlizando WSL (Windows Subsystem for Linux). Neste caso, execute o seguinte comando dentro do seu Linux:

    `sudo sysctl -w vm.max_map_count=262144`

    *Você pode aumentar ou diminuir o valor, caso achar necessário.*

3. Caso você tenha feito o deploy do projeto em um cluster Kubernetes, se você tentar acessar a aplicação por meio do IP entregue pelo **service** do Kubernetes, vai notar que o mapa não aparece e um erro **GeolocationPositionError** é exibido no console do navegador com uma ***message: "Only secure origins are allowed (see: https://goo.gl/Y0ZkNV)".*** Isso acontece porque não temos nada de TLS configurado, e, por padrão, a API do Google Maps só aceita ser acessada por meio de uma **origem https**. Para testes, podemos contornar isso da seguinte maneira:

    * Acesse no seu navegador **chome://flags**;
    * Encontre a opção **Insecure origins treated as secure**, habilite-a e insira o endereço de IP que você obteve pelo service do Kubernetes.
    *Obs: Você pode consultar o endereço de IP do seu service consultando: kubectl get services*;
    * Reinicie o navegador e tente acessar novamente.