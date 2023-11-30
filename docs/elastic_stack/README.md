## Configurando Elastic Stack e Kibana

Como já temos um connector do Kafka Connect mandando os dados do tópico do Kafka para o Elastic Search, agora temos que configurar a maneira como o Elastic Search vai mapear os dados e, por fim, as dashboards do Kibana.

Vamos lá:

0. No Kafka Control Center, acesse **Connect > Add connector** no menu lateral do seu cluster (Cluster 1, provavelmente) e vá em **Upload connector config file**. Selecione, então, o arquivo **elasticsearch.properties** que está no projeto. 
1. Acesse o menu lateral e navegue para **Management > Stack Management > Index Management**. Então verifique se os índices foram criados de acordo com os tópicos que temos no Kafka;
2. Se possível, antes de prosseguir, publique alguma mensagem no Kafka (exemplo `{"routeId": "1", "clientId":"a"}`), apenas para que tenhamos dados para criar os index patterns;
3. Navegue até **Management > Stack Management > Index Patterns**, clique em *Create index pattern* e crie um index pattern para cada tópico (com o nome do respectivo tópico) que temos no Kafka;
4. Neste momento, se você navegar para **Analytics/Discover**, você vai notar em **Available fields** que o campo **position** está sendo tratado como números apenas, não como coordenadas.
5. Para solucionar este problema, vamos navegar para **Management > Dev Tools**. Neste console podemos fazer requisições para o Elastic Search. Nele, vamos criar um índice que é como se fosse um "schema" para mapear as informações que queremos no Kibana. Além disso, vamos adicionar um timestamp de quando o registro foi criado. Para isto, cole no console e execute as seguintes requisições:
```
PUT route.new-position
{
  "mappings": {
    "properties": {
      "clientId": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "routeId": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "timestamp": {
        "type": "date"
      },
      "finished": {
        "type": "boolean"
      },
      "geoJsonPosition": {
        "type": "geo_point"
      }
    }
  }
}

PUT route.new-direction
{
  "mappings": {
    "properties": {
      "clientId": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "routeId": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "timestamp": {
        "type": "date"
      }
    }
  }
}
```
*Obs: Caso queira deletar estes dados, você pode executar as seguintes requisições:*
```
POST route.new-position/_delete_by_query?conflicts=proceed
{
  "query": {
    "match_all": {}
  }
}

POST route.new-direction/_delete_by_query?conflicts=proceed
{
  "query": {
    "match_all": {}
  }
}
``` 
6. Se você observar mais uma vez no **Discover**, vai perceber que o **timestamp** está sendo utilizado como um campo comum. Mas nos queremos que ele seja a referência de data e hora no Kibana. Para isso, novamente vamos ter que deletar os índices em **Stack Management > Index Patterns**. Agora, vá em **Create index pattern** novamente, mas, antes de prosseguir, selecione o campo **timestamp** em **Select a primary time field for use with the global time filter**. Isso vai fazer com que este campo seja usado para nortear toda a data dos seus dados. Faça isso para ambos os índices (route.new-direction e route.new-position).
Se você acessar o **Discover** agora, vai ver que automaticamente um gráfico já estará sendo gerado com os dados agrupados de acordo com o campo de data referência.


### Criando o dashboard

Para criar o dashboard, teremos que criar **visualizações**. Para isso, no menu, acesse **Analytics > Visualize**.
Agora, clique no botão **Create new visualization** e vá em **Lens** (uma das formas mais fáceis de se criar visualizações).

1. Quantidade de corridas
    * Selecione o index **route.new-direction** e arraste para o painel do centro a opção **Records**;
    * Troque o tipo **Stacked bar** por **Metric**;
    * Clique em **Count of records** e mude para o nome desejado. Por exemplo, "Corridas";
    * Clique em **Save** e dê um nome para a visualização.
2. Browsers conectados (dentro de um espaço de tempo)
    * Em **Available fields**, arraste o campo **clientId.keyword** para o painel central;
    * Mude de **Stacked bar** para **Metric**;
    * Mude o *Display name* (Browsers conectados);
    * Clique em **Save** e dê um nome para a visualização.
3. Quantidade de corridas por rota
    * Em **Available fields**, arraste o campo **routeId.keyword**;
    * Altere o formato de **Stacked bar** para **Data table**;
    * Mude o *Display name* do **Top values of routeId.keyword** (Rotas);
    * Mude o *Display name* do **Count of records** (Quantidade de corridas);
    * Clique em **Save** e dê um nome para a visualização (Quantidade de corridas por rota).
4. Quantidade de corridas por rota (Gráfico de pizza)
    * Em **Available fields**, arraste o campo **routeId.keyword**;
    * Altere o formato de **Stacked bar** para **Pie**;
    * Mude o *Display name* (Rotas);
    * Clique em **Save** e dê um nome para a visualização (Gráfico de rotas).
5. Mapa
    * Agora, ao invés de ir selecionar uma visualização do tipo **Lens**, vamos criar uma do tipo **Maps**;
    * Clique em **Add layer** e selecione **Documents**;
    * Em **Index patterns**, selecione **route.new-position** e clique em **Add layer**;
    * Em **Source details > Layer settings** da layer criada, atribua coloque o **Zoom levels** como **10 -> 24**;
    * Também em **Source details > Layer settings**, atribua um **Name** (Rota 1);
    * Agora, em **Source details > Tooltip fields**, clique em **Add** e selecione o **routeId**;
    * Em **Source details > Filtering**, clique em **Add filter** e passe a seguinte query: `routeId.keyword : "1"`;
    * Em **Source details > Layer Style**, selecione as cores em **Fill Color** e **Border color**;
      * Rota 1: #54B399 e #41937C;
      * Rota 2: #D36086 e #C83868;
      * Rota 3: #6092C0 e #4379AA.
    * Por fim, clique em **Save & close**
    *Obs: Para focalizar o mapa em uma rota específica, clique na rota desejada em **Layers** e clique em **Fit to data***
    * Antes de finalizar, faça a mesma coisa para as outras duas rotas (2 e 3), devidamente ajustando o **Name**, o **Filtering**, **Fill color** e **Border color** para cada uma delas. *Obs: Você pode clicar com o botão direito em uma layer e ir em Clone layer para facilitar*;
    * Agora, para finalizar, clique em **Save** no canto direito superior e dê um nome para o mapa (Mapa).

**Hora da Dashboard**

* Para criar uma dashboard, navegamos no menu lateral para **Analytics > Dashboard** e clicamos em **Create new dashboard**;
* Para adicionar as visualizações criadas, clique em **Add from library** e adicione todas as suas visualizações na ordem e tamanhos que desejar;
* Por fim, clique em **Save** no canto direito superior e dê um nome para a dashboard (Dashboard de corridas).

*Obs: Se você acessar a dashboard agora, verá no canto direito superior da tela que ela está exibindo os dados dos últimos 15 minutos por padrão. Você pode alterar este valor e a dashboard irá mudar de acordo com aquele campo **timestamp**.*