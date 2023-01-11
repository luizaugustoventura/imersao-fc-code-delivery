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