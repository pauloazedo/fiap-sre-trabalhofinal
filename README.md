## fiap-sre-trabalhofinal
# Monitorando Sistemas Operacionais utilizando Docker, Telegraf, InfluxDB e Grafana

Este é o trabalho de conclusão da matéria de SRE do curso DevOps Engineering da FIAP. como Grafana, Docker e Telegraf com Influxdb.

Usaremos o Docker para uma rápida implementação em qualquer ambiente Docker, uma base de dados no Influxdb para armazenar nossas métricas coletadas e o Telegraf como agente no sistema monitorado remoto, e para visualizarmos os dados coletados através de Dashboards, utilizaremos o Grafana.

Este passo a passo foi desenvolvido e testado em abiente Linux, distribuição CentOS 7 com a instalação padrão do Docker.

Vamos lá!

## Instalação e configuração do InfluxDB e Grafana no Docker

Criar um diretório de trabalho:

    $ mkdir /opt/fiap-sre-trabalhofinal && cd /opt/fiap-sre-trabalhofinal

Efetuar um git clone no projeto https://github.com/pauloazedo/fiap-sre-trabalhofinal.git

    $ git clone https://github.com/pauloazedo/fiap-sre-trabalhofinal.git

Pronto, já temos todos os arquivos que iremos precisar para executar todo o projeto.

Criaremos uma rede para os nossos serviços chamada monitoring e criar um volume externo para salvarmos os dados e configurações das aplicações, prevenindo a perda de dados quando os containers forem reiniciados. Utilizar uma rede própria é uma boa prática para isolar os conteiners que pertencem ao mesmo projeto:

    $ docker network create monitoring

    $ docker volume create grafana-volume

    $ docker volume create influxdb-volume

Verifique se tudo foi criado corretamente:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/1.png?raw=true)

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/2.png?raw=true/)

Como podemos ver, a rede e os volumes foram criados corretamente.

Agora iremos preparar os parâmetros do Influxdb, para isso iremos executar o container com alguns parametros para criação da database e usuários:

    $ docker run --rm \    
    -e INFLUXDB_DB=telegraf -e INFLUXDB_ADMIN_ENABLED=true \    
    -e INFLUXDB_ADMIN_USER=admin \    
    -e INFLUXDB_ADMIN_PASSWORD=123456 \    
    -e INFLUXDB_USER=telegraf -e INFLUXDB_USER_PASSWORD=123456 \    
    -v influxdb-volume:/var/lib/influxdb \    
    influxdb /init-influxdb.sh

Obs: Executaremos este container com o parâmetro –rm, isso irá criar somente os configs e remover o container depois.

Tudo pronto, vamos iniciar o nosso sistema de monitoramento através da execução do docker-compose:

    $ docker-compose up -d

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/3.png?raw=true)

Agora com os containers criados e iniciados, o sistema de monitoramento está preparado para ser configurado. Nós expomos algumas portas no docker-compose, a **8086** http para a API do Influxdb e a porta **3000** http para a UI do Grafana.

Acessar o endereço do servidor na porta **3000** no browser e logar no Grafana:

> Usuário: admin
> Senha: admin

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/4.png?raw=true)

Ele irá pedir para trocar a senha, podemos alterar ou clicar em em **Skip**:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/5.png?raw=true/)

Agora vamos adicionar um Datastore, selecione **ADD Datasource**:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/6.png?raw=true)

Selecione o **InfluxDB**:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/7.png?raw=true)

Preencha com os dados que utilizamos na criação das databases e o endereço da API do InfluxDB:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/8.png?raw=true)

Clique em **Save and Test**, se recebermos a mensagem abaixo é porque o Grafana conseguiu se conectar ao InfluxDB:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/9.png?raw=true)

Agora que configuramos nosso InfluxDB como datasource no Grafana, iremos utilizar um dashboard pronto disponível no **GrafanaLabs** que contém os parâmetros mais utilizados e anote o código [914](https://grafana.com/grafana/dashboards/914):

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/10.png?raw=true)

Clique em **New Dashboard**:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/11.png?raw=true)

Clique em **Create** e então **Import**:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/12.png?raw=true)

Insira o código **914** e clique em Load:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/13.png?raw=true)

Selecione o datasource **InfluxDB** que criamos anteriormente e clique em import:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/14.png?raw=true)

Aqui finalizamos a criação do datasource do InfluxDB.

## Instalação do client Telegraf no servidor a ser monitorado

Efetuar a instalação do client Telegraf conforme o procedimento no site oficial de acordo com a distribuição que esteja utilizando:

[https://docs.influxdata.com/telegraf/v1.7/introduction/installation/](https://docs.influxdata.com/telegraf/v1.7/introduction/installation/)

No nosso caso, estamos utilizando o CentOS:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/15.png?raw=true)

Agora vamos editar o config do Telegraf do servidor a ser monitorado e adicionar os parâmetros necessários para ele enviar informações para a API do InfluxDB:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/16.png?raw=true)

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/17.png?raw=true)

Após alterar as configurações, reiniciar o serviços do telegraf:

    $ systemctl restart telegraf

Pronto! Temos agora um dashboard com vários dados coletados em tempo real do servidor que estamos monitorando de forma estruturada. Neste exemplo, o host monitorado é o mesmo que está instalado o docker rodando os containers do Grafana e do InfluxDB:

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/18.png?raw=true)

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/19.png?raw=true)

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/20.png?raw=true)

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/21.png?raw=true)

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/22.png?raw=true)

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/23.png?raw=true)

![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/24.png?raw=true)
![](https://github.com/pauloazedo/fiap-sre-trabalhofinal/blob/master/imagens/25.png?raw=true)
