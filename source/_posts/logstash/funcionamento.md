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
 - logstash 

tags:
---

Responsável por coletar dados de fontes externas (Logs & Files, Web Apps, Metrics, Data Stores, SGBD, APIs e DataStream), manipular/processar estes dados (pipeline) e submetê-los aos agregadores como o Elasticsearch. Funciona como ferramenta de __ETL__ permitindo DataFlow pipeline do ELK.

<a href="https://www.elastic.co/pt/webinars/getting-started-logstash?baymax=rtp&storm=ribbon-1&elektra=products-logstash&iesrc=ctr" target="_blank">Vídeo Explicativo sobre o Logstash</a>

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

# Arquivo .conf Base
## Peças do .conf
Os arquivos de configuração do logstash são divididos em 3 partes básicas:
* __input__: definir qual ou quais plugins serão usados para a entrada dos dados, por exemplo o __beats__, um __driver JDBC__ (ex.: MySQL, Orcale...), __HTTP__ (API REST);
* __fitler__: define os plugins que manipularão os dados e quais manipulações cada plugin deste efetuará;
* __output__: definir os plugins para onde os dados, após processados serão enviados.

## Arquivo .conf de exemplo
Este arquivo configura para que o logstash pegue dados do beats (logger do ELK) e transforme os caracteres das mensagens todas em caixa baixa (lowcase) e a saída será inserida no elasticsearch sem configuração, assim ele subentenderá que o elastic está rodando na mesma máquina (localhost e porta default).

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

# Input Plugins, Filter Plugins & Output Plugins

## (INPUT) JDBC plugin
O JDBC é um plugin do logstash para que este se conect a bases de dados utilizando adaptadores JDBC (.jar), assim sendo é necessário ter os drivers que estão diponíveis para download neste projeto e colocá-lo no diretório a ser configurado no .conf para que o logstash o use para coletar dados do SGBD:

```conf  base_development.conf
input {
  jdbc {
    jdbc_user => "root"
    jdbc_password => ""
    jdbc_connection_string => "jdbc:mysql://localhost:3306/base_development"
    jdbc_driver_library => "/drivers/mysql-connector-java-5.1.47-bin.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    statement => "SELECT * from base_development.projects"
  }
}

output {
    elasticsearch {
        "hosts" => "localhost:9200"
        "index" => "projects"
    }
    stdout { codec => json_lines }
}
```

Neste exemplo alguns pontos a serem ressaltados são:
1. Este .conf não efetuou nenhum filtro, porém seria interessante fazer ao menos as marcações do campo timestamp
1. No campo __input__ só possui um filho/adaptador chamado __jdbc__ e dentro deste os attrs preenchidos foram usuário, senha, string de conexão, localização do driver na máquina (__jdbc_driver_library__), classe do driver a ser chamada, no caso é o .class do .jar (__"com.mysql.jdbc.Driver"__) e por fim o statement que é o SQL a ser executado na base de dados de destino;
1. No output, como é o localhost não seria necessário configurar o hosts, porém assim fica mais visível como deve ser a configuração quando rodando em multiplas máquinas e com acessos externos. O __"index"__ é o ID do projeto dentro do elasticsearch para facilitar a pesquisa no Kibana e também para organizar os datasources dentro do elasticsearch para evitar conflito de dados.

## (INPUT) Beats plugin
O beats é mais uma peça Standalone do pacote ELK. <a href="/elk/install" target="_blank">Para instalar o Beats no MacOS siga os passos de instalação</a>.
Para executar o beats considerando a instalação pelo brew é: 
```shell script
    $ /usr/local/Cellar/metricbeat-full/version/bin/metricbeat -f commands_file.yml
```

Exemplo de código do arquivo commands_file.yml do beats:
```yaml commands_file.yml
filebeat.prospectors:
- type: log
  enabled: true
  paths:
    ./file_name.log

output.logstash:
    hosts: ["localhost:5044"]  
```

Este arquivo yml indica ao beats que ele deve prospectar dados de um arquivo do tipo log localizado em ./ e jogar esta saída para o logstash hospedado em localhost:5044.
__Obs.:__ o beats só deve ser usado após um __.conf__ ser configurado no logstash para que este fique apto a processar

## (FILTER) Grok -> Beats
No arquivo __inputs_grok_sample.conf__ abaixo ele apenas fica apto esperando que o beats lhe envie dados como descrito no item anterior. Assim sendo, primeiro é necessário rodar o logstash com o .conf deste item para que com esta configuração disponibilizada e ativa ele seja capaz de receber o beats.
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

## (OUTPUT) elasticsearch & stdout
No arquivo do __inputs_grok_sample.conf__ os outputs definidos foram: __elasticsearch__ & __stdout__ sendo o elasticsearch aonde este dado será armazenado e o stdout é apenas um "log" com pretty print do evento que foi executado para haver um feedback visual em nível de desenvolvimento/debug.

Exemplo de __output__ do pretty debug do __stdout__ codec rubydebug:
```json output_stdout_codec_rubydebug
{
  "prospector": {
    "type": "log"
  },
  "tags": ["beats_input_codec_plain_applied"],
  "@version": "1",
  "source": "/Users/../../file_name.log",
  "@timestamp": 12173021,
  "line": "Lorem ipsum lat",
  "beat": {
    "name": "ElasticMBP.local",
    "hostname": "ElasticMBP.local",
    "verison": "6.2.4"
  }
}
```

Exemplo de __output__ inserido no __elasticsearch__ visível a partir do __kibana__, mostrando o mesmo log do print do __stdout__:
![Kibana mostrando o log de processamento do Beats importado no Elasticsearch via Logstash](/elk/images/kibana_logstash_elasticsearch_beats_log_import.png)


# Setup Drivers

## MySQL

```shell script
    $ cp drivers/mysql-connector-java-5.1.47-bin.jar /db_drivers
```

## Oracle

```shell script
    $ cp drivers/ojdbc6.jar /db_drivers && cp drivers/ojdbc7.jar /db_drivers &&  
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
