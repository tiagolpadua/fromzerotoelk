# fromzerotoelk
Passos para criar uma stack Elasticsearch, Logstash e Kibana

# Docker
https://docs.docker.com/get-started/

Docker Tutorial - What is Docker & Docker Containers, Images, etc? (7min)

https://www.youtube.com/watch?v=pGYAg7TMmp0

https://www.digitalocean.com/community/tutorials/como-instalar-e-usar-o-docker-no-ubuntu-16-04-pt

# ELK
An Introduction to the ELK Stack (Now the Elastic Stack)

https://www.elastic.co/webinars/introduction-elk-stack (36min)

What Is ELK Stack | ELK Tutorial For Beginners | Elasticsearch Kibana | ELK Stack Training | Edureka

https://www.youtube.com/watch?v=MRMgd6E9AXE (41min)

<!--
Não utilizado
https://github.com/deviantony/docker-elk
-->

Elasticsearch, Logstash, Kibana (ELK) Docker image
https://hub.docker.com/r/sebp/elk/
https://elk-docker.readthedocs.io/

# Filebeat
https://www.elastic.co/products/beats/filebeat

https://www.elastic.co/videos/getting-started-with-filebeat?baymax=rtp&storm=beats&elektra=product&iesrc=ctr (4min)

# Logstash

# Elastic Search

# Kibana

# Passo a Passo

## Máquina que irá executar a pilha ELK
```bash
docker pull sebp/elk
docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk sebp/elk
docker start elk (-a para attached)
docker exec -it elk /bin/bash
docker stop elk
docker ps
docker images 
docker save -o elk_image.docker sebp/elk
docker load elk_image.docker
```

* 5601 (Kibana web interface) - http://localhost:5601
* 9200 (Elasticsearch JSON interface) - Listar todos os logs: http://localhost:9200/_search?pretty
* 5044 (Logstash Beats interface, receives logs from Beats such as Filebeat – see the Forwarding logs with Filebeat section).

### Remover necessidade de ssl temporariamente e incluir o filtro grok para apache:
/etc/logstash/conf.d/02-beats-input.conf
```
input {
  beats {
    port => 5044
  }
}
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
}

```


## Máquina que irá gerar os logs

### Vagrant trusty
Será utilizado uma imagem Vagrant - https://app.vagrantup.com/ubuntu/boxes/trusty64

```bash
cd
mkdir trusty64
vagrant init ubuntu/trusty64
```
Editar arquivo Vagrantfile para fazer foward da porta 8080:
-> config.vm.network "forwarded_port", guest: 80, host: 8080

```bash
vagrant up
vagrant ssh
```

### Apache2
Instalar o apache2 (https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04):
```bash
sudo apt-get update
sudo apt-get install apache2
sudo apachectl start
```

No host, testar o endereço http://localhost:8080, deve exibir a página inicial do apache.

### Filebeat
```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.0.1-amd64.deb
sudo dpkg -i filebeat-6.0.1-amd64.deb
sudo /etc/init.d/filebeat start
sudo /etc/init.d/filebeat stop
```

Arquivo de configuração: /etc/filebeat/filebeat.yml
```yml
filebeat.prospectors:
- type: log
  enabled: true
  paths:
    - /var/log/apache2/*.log
output.logstash:
  enabled: true
  hosts: ["elk:5044"]
```
Onde 'elk' é o ip da máquina que receberá os logs

Gerando tráfego:
```bash
sudo apt-get install apache2-utils
ab -n 500000 -c 10 http://localhost:8080
```
