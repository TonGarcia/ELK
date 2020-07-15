---
title: ElasticSearch + Logstash + Kibana & AllStack
date: 2019-03-11 12:09:42
---
Esta Wiki tem como objetivo ser um MiniGuia de [instalação](/elk/instalacao), [manutenção](/elk/manutencao), [escala](/elk/escala) e [desenvolvimento de Dashboards](/elk/desenvolvimento) utilizando a Stack ELK.

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
* [__Beats__](https://www.elastic.co/pt/beats/): 
    * "Micro Serviços" de coleta de dados em nível de sistema de forma leve. Podem ser containers ou implantados como funções e centralizam os dados no Elasticsearch. Se precisar de mais poder de processamento eles encaminham os dados ao Logstash para transformação e análise;
    * Nativamente utilizado para coleta de logs de servidores/containers/IoT. São micro serviços que rodam na ponta coletando dados e enviando ao Elasticsearch.
* [__Logstash__](https://www.elastic.co/pt/logstash):
    * Responsável por coletar dados de fontes externas (Logs & Files, Web Apps, Metrics, Data Stores, SGBD, APIs e DataStream), manipular/processar estes dados (pipeline) e submetê-los aos agregadores como o Elasticsearch;
    * Funciona como ferramenta de __ETL__ permitindo DataFlow pipeline do ELK;
    * PlugIns:
        * __INPUTS PLUGINS__:
            * Passivos (fontes de dados enviam dados ao logstash): Beats, TCP, UDP, HTTP;
            * Ativos (logstash busca dados na fonte): JDBC, HTTP Poller.
        * __FILTER PLUGINS__:
            * são plugins usados para estruturar, normalizar, transformar e enriquecer dados, por exemplo acrescentar geolocalização através do dado do GeoIP.
        * __OUTPUTS PLUGINS__: 
            * codecs que serializam o dado para o data source de destino, entre eles: __Elasticsearch__, TCP, UDP, HTTP entre outros Stores e/ou Queues.
    * Filas (Queues):
        * Persistent Queues: permite persistir pilha de processamento agendando tarefas futuras evitando perdas;
        * Dead Letter Queues: pilhas standby para uso posterior e/ou baixa prioridade que ficam local no logstash sem ir para local de armazenamento.
    * Pipelines: múltiplos e __com condicionais__ configuráveis.
* __Elasticsearch__: 
    * Engine de Search e Analytics 
    * Busca/filtragem de dados muito rápida
* __Kibana__:
* __X-Pack__:
    * Extensão ELK que inclui sistema de login/segurança para controle de acesso de usuário ao ELK, alertas configuráveis (disparo de e-mails/mensagens), monitoring (monitoramento do uso de recursos e atividade do ELK), reporting (exportar PDF dos dashboards), gráficos personalizados e complexos (graph) e aprendizado de máquina (IA).


[Preços Infra ELK Cloud (mín.: US$ 16/mês e top.: US$ 22/mês)](https://www.elastic.co/pt/pricing/) vs [Preço Infra ELK GCP - Bitnami (US$ 24,75/mês)](https://console.cloud.google.com/marketplace/details/bitnami-launchpad/elk)

[Link de Acesso ao Google Trends abaixo.](https://trends.google.com.br/trends/explore?geo=BR&q=elasticsearch,logstash,kibana,%2Fm%2F052hvb,powerbi)

![Tendências de ferramentas (Google Trends)](/elk/images/google-trends-elk.png)

# Arquitetura - DEV
![Estrutura ELK - DESENVOLVIMENTO](/elk/images/elk-arch-development.png)

# Arquitetura - PROD
![Estrutura ELK - PRODUÇÃO](/elk/images/elk-arch-production.png) 
