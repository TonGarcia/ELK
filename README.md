# ELK
ELK Stack Study. Acesso usuário: https://tongarcia.gitlab.io/elk/

# Local Access
1. Run: ```hexo server```
1. Access: http://localhost:4000/elk

# Logstash
Check logstash pipeline perform
```shell script
    $ /usr/local/Cellar/logstash/version/bin/logstash -f commands_file.conf
```

# Elastic Search
Check logstash data in elastic search endpoint (default limit(size)=10)
```shell script
    $  curl 'http://localhost:9200/projects/_search?size=1000&pretty=true'
```

# Kibana
Check kibana dashboard
```shell script
    $  curl 'http://localhost:9200/projects/_search?size=1000&pretty=true'
```

# Repositories
1. GitHub = https://github.com/TonGarcia/ELK
1. GitLab (pages/blog) = https://gitlab.com/TonGarcia/elk

# Install
To install ELK using brew correctly and workarounds for possible errors [check it tutorial](/elk/install)
