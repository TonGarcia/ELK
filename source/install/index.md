---
title: Instalação da Stack ELK
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
 - elasticsearch 
 - [Importação de Dados, Bancos de Dados (SGBD)]

tags:
---

A Stack ELK deve preferencialmente ser instalada na máquina através de pacotes de gerenciamento de dependências como o apt-get, homebrew, chocolate e etc. Porém pode ser instalada através do Docker.

# MacOS
Passos para instalação no MacOS:

## Instale o HomeBrew

```shell script
   $ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)" 
```
## Instale os pacotes ELK separadamente

```shell script
   $ brew install logstash-full && brew install elasticsearch-full && brew install kibana-full
```

## Inicialização dos Serviços

```shell script
   $ brew services start logstash-full && brew services start elasticsearch-full && brew services start kibana-full
```

## Verificar Serviços Ativos

```shell script
   $ brew services list
```

## Setup Drivers
  
### MySQL

```shell script
  $ cp drivers/mysql-connector-java-5.1.47-bin.jar /db_drivers
```


# Linux

# Windows

# Docker
