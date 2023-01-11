## Informações e instruções importantes

Para que o Docker possa resolver o **host.docker.internal**, é necessário incluir o endereço **127.17.0.1** no **/etc/hosts** da sua máquina.

*Obs: No Windows, você vai encontrar este arquivo no seguinte diretório **C:\Windows\System32\drivers\etc***

A seguinte linha deve ser inserida ao final do arquivo:

`172.17.0.1 host.docker.internal`

*Obs: Ainda pode ser que você venha a ter um problema relacionado ao consumo de memória pelo Docker, caso esteja utlizando WSL (Windows Subsystem for Linux). Neste caso, execute o seguinte comando dentro do seu Linux:*

`sudo sysctl -w vm.max_map_count=262144`

*Você pode aumentar ou diminuir o valor, caso achar necessário.*