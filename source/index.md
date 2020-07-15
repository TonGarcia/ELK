---
title: ElasticSearch + Logstash + Kibana & AllStack
date: 2019-03-11 12:09:42
---
Esta Wiki tem como objetivo ser um MiniGuia de [instalação](/wiki/instalacao), [manutenção](/wiki/manutencao), [escala](/wiki/escala) e [desenvolvimento de Dashboards](/wiki/desenvolvimento) utilizando a Stack ELK.

Links iniciais úteis: 
1. [INTRO ELK](https://www.elastic.co/pt/elasticon/tour/2020/sao-paulo/opening-keynote?ultron=all-elastic&hulk=cpc&blade=linkedin)
1. [Start Guide](https://www.elastic.co/pt/start)
1. [Downloads](https://www.elastic.co/pt/downloads/)
1. [Elastic Observability](https://www.elastic.co/pt/observability)
1. [Elastic Segurança](https://www.elastic.co/pt/security)
1. [Elastic Stack](https://www.elastic.co/pt/elastic-stack)
1. [Elastic Cloud](https://www.elastic.co/pt/cloud/)
1. [Elastic Orquestração](https://www.elastic.co/pt/ece)
1. [Elastic Kurbenets](https://www.elastic.co/pt/elastic-cloud-kubernetes)

Iniciar os serviços via __brew__:
```shell script
    $ brew services start logstash-full && brew services start elasticsearch-full && brew services start kibana-full
```

Acesse o kibana no navegador: http://localhost:5601/

Videos intro:
1. https://www.youtube.com/watch?v=Bb3g8xk0Cys
1. https://www.youtube.com/watch?v=j8X8EG-jaaA
1. https://www.youtube.com/watch?v=LapNa2l-7VA

# Relevância e Funcionamento
Apesar do Kibana ser a parte mais externa da Stack ELK, sendo o front-end (a qual o usuário final interage), o Google Trends mostra que os elementos separados do ELK são bastante procurados para os objetivos que possuem:
* __Beats__: 
* __Logstash__:
* __Elasticsearch__:
* __Kibana__:

![Tendências de ferramentas (Google Trends)](/wiki/images/google-trends-elk.png)

# Arquitetura - DEV
![Estrutura ELK - DESENVOLVIMENTO](/wiki/images/elk-arch-development.png)

# Arquitetura - PROD
![Estrutura ELK - PRODUÇÃO](/wiki/images/elk-arch-production.png) 
