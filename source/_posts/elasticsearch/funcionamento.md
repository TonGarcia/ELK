---
title: Funcionamento do Logstash
date: 2019-08-04 09:17:21
toc: true
sidebar:
    left:
        sticky: true
widgets:
    -
        type: toc
        position: left

category:
 - elasticsearch

tags:
---

<a href="https://www.youtube.com/watch?v=zRExXSESukI&feature=share" target="_blank">Vídeo Explicativo sobre o ElasticSearch</a>

# TODO remover tudo



Responsável por coletar dados de fontes externas (Logs & Files, Web Apps, Metrics, Data Stores, SGBD, APIs e DataStream), manipular/processar estes dados (pipeline) e submetê-los aos agregadores como o Elasticsearch. Funciona como ferramenta de __ETL__ permitindo DataFlow pipeline do ELK.

<a href="https://www.elastic.co/pt/webinars/getting-started-logstash?baymax=rtp&storm=ribbon-1&elektra=products-logstash&iesrc=ctr" target="_blank">Vídeo Explicativo</a>

TODO: parei esse vídeo em 19 minutos

<!-- more -->

A configuração do Logstash passa por um arquivo no qual é informado os plugins e as tasks atribuídas em cada parte (input, filter, output):

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

A fila default é a Memory Queue, na qual o logstash salva os eventos a serem disparados em memória. Caso ocorra algum erro fatal no logstash ou no computador os dados podem ser perdidos. Ao configurar uma Persistent Queue significa que os eventos serão salvo em disco e caso aconteça algum problema será possível retomar as execuções após se recuperar do problema. O Logstash garante que ao menos 1 entrega do processamento será feito, podendo ter mais de 1. __Para evitar inserção duplicada é importante que crie IDs para os eventos__. 

# Arquivo Config Base
Este arquivo configura para que o logstash pegue dados do beats e transforme as mensagens todas em caracteres lowercase e a saída será inserida no elasticsearch sem configuração, assim ele subintenderá que está rodando tudo na mesma máquina.

```conf minimal_base.conf
    input {
        beats { port => 5403 }
    }

    //implicit queue
    filter {
        mutate { lowercase => ["message"] }
    }

    output {
        elasticsearch {}
    }
```

# Input Plugins & Filter Plugins

## BeatsInput Plugin com GrokFilter Plugin
1. grok plugin: usado para processar dados de log e trabalha com expressões regulares com variáveis que este consegue capturar no texto
    1. no arquivo abaixo o grok está fazendo:
        1. identifica o padrão __TIMESTAMP_ISO8601__ e aloca o valor deste na variável __timestamp_string__
        1. depois ele considera um espaço (caso tire o espaço do arquivo abaixo o arquivo processado precisaria seguir esse padrão)
        1. o __GREEDYDATA__ significa consumir o resto da linha e aloca na variável __line__
1. date plugin:
    1. verifica se a variável __timestamp_string__, preenchida pelo grok, está no formato __ISO8601__
    1. irá criar um objeto do tipo date usando o valor coletado após essa validação e jogará em uma variável campo padrão do logstash chamado @timestamps ... (parser/typecast). Esse campo é reconhecido pelo Kibana por padrão.
1. mutate plugin:
    1. como os campos já foram coletados pelo grok e processado pelo date eles não são mais necessários, geraria inserção de lixo no elasticsearch
    1. assim sendo usou-se o mutate para remove esses campos

```conf inputs_grok_sample.conf
    input {
        beats { port => 5044 }
    }

    //implicit queue
    filter {
        grok {
            match => [
                "message", "%{TIMESTAMP_ISO8601:timestamp_string} %{GREEDYDATA:line}
            ]
        }
        date { match => ["timestamp_string", "ISO8601"] }
        mutate { remove_field => ["message", "timestamp_string"] }
    }

    output {
        elasticsearch {}
        stdout {
            codec => rubydebug
        }
    }
```

# Output Plugins

No arquivo do inputs_grok_sample.conf o output foi para o elasticsearch padrão com stdout codec rubydebug que é utilizado para fazer pretty print (impressão bonita e legível) do evento ocorrido para ter certeza de que o processamento foi feito com sucesso.

# Setup Drivers

## MySQL

```shell script
    $ cp drivers/mysql-connector-java-5.1.47-bin.jar /db_drivers
```

# Executando o Logstash

## MacOS

```shell script
    $ /usr/local/Cellar/logstash/version/bin/logstash -f commands_file.conf
```

# Bug Fix

## Fatal
Em caso de erro fatal é becessário matar o processo:

```shell script
    $ killall logstash
```
