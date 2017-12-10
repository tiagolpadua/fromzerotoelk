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
## Máquina que irá gerar os logs
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

Instalar o apache2 (https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04):
```bash
sudo apt-get update
sudo apt-get install apache2
sudo apachectl start
```

No host, testar o endereço http://localhost:8080, deve exibir a página inicial do apache.

Ainda falta configurar o Filebeat, mas é melhor configurar após a máquina do ELK estar funcionando

## Máquina que irá executar a pilha ELK
```bash
docker pull sebp/elk
docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk sebp/elk
docker start elk (-a para attached)
docker stop elk
docker ps
docker images 
docker save -o elk_image.docker sebp/elk
docker load elk_image.docker
```

* 5601 (Kibana web interface) - http://localhost:5601
* 9200 (Elasticsearch JSON interface) - Listar todos os logs: http://localhost:9200/_search?pretty
* 5044 (Logstash Beats interface, receives logs from Beats such as Filebeat – see the Forwarding logs with Filebeat section).

