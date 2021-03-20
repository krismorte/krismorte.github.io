---
title: "Documentando bancos com Schemaspy"
date: "2020-04-12T18:55:33Z"
tags: ["java","database"]
---

Quando penso em documentação de banco de dados lembro-me de ferramentas centralizadoras onde tudo relacionado ao banco de dado parte do DBA, design, alterar tipo de coluna era necessário abrir um ticket pro DBA e coisas nesse sentido. Num mundo sem migrations e CI\CD fazia muito sentido centralizar tudo na mão do cara especializado, bancos imensos com milhares de tabelas alguns teras de dados e atendendo as vezes mais de um sistema , nos fazia necessário de um [ditador benevolente](https://git-scm.com/book/en/v2/Distributed-Git-Distributed-Workflows).

Nesse novo mundo de microsserviços, CI\CD e migrations todo o poder de decisão sobre o design foi passado pra aplicação, bancos cadas vez menores e mais focados em um único sistema e livres estrutura que contenha regra de negocios. Nesse novo mundo ainda vale a pena ter essa ferramenta centralizadora pra controlar o banco de dados? Bem nesse ponto encontrei [SchemaSpy](http://schemaspy.org/) e realmente acredito que não.

SchemaSpy é uma aplicação feita em Java que escaneia todo o seu banco de dados e gera um web site estatico contendo todos as tabelas, views, funções e relaçãoes entre elas além do script de criação de alguns. Imagina "navegar" pelo seu banco de dados de maneira totalmente visual com direito a gráficos e analise simples de anomalias. Sim esse é o cara.

Vamos por em pratica isso, será necessário ter instalado o graphviz, java e baixar a ultima versão do SchemaSpy e drive jdbc do banco que você quer documentar, simples assim. O uso basicamente se dar por linha de comando conforme explicado no site:

`java -jar schemaspy.jar -t mssql05 -dp C:/sqljdbc4-3.0.jar -db DATABASE -host SERVER -port 1433 -s dbo -u USER -p PASSWORD -o DIRECTORY`

do comando destaco a opção `-t` que diz que tipo de banco de dados, você pode conferir todos suportados [aqui](https://github.com/schemaspy/schemaspy/tree/master/src/main/resources/org/schemaspy/types), `-dp` que é onde você indica o drive jdbc ou o diretório onde o drive está e `-o` que é o diretório onde será gerado o site estativo. Temos o `-s` pra indicar o schema, pro sql server não se faz obrigatório mas pro mysql sim e bem útil no oracle.

Até aqui tudo bem né? Bem, não pra mim, ao encontrar essa solução me ajudou muito, mas logo em seguida me veio a cabeça que meu servidor de banco dados tinha mais de 40 banco de dados e o schemaspy documenta apenas um banco por vez. Eis que me surgiu a ideia de englobar o schemaspy numa solução minha pra documentar ao menos um server por vez e a surgiu o [Database Diagrams](https://github.com/krismorte/database-diagrams). Sendo bem direto se trata de uma imagem docker com nginx pra exibir o sites gerados, cron job pra agendar a execução e uma pequena aplicação java que configura e gerencia as seguidas execuções do schemaspy. Como usar? A imagem gerada ja se encontra no [docker huh](https://hub.docker.com/repository/docker/krismorte/databasediagrams)

`docker pull krismorte/databasediagrams:2.0`

Se quiser testar primeiro clonar o repositório git e executar o docker-compose lá

```
git clone https://github.com/krismorte/database-diagrams.git
cd database-diagrams
docker-compose up
```

Subindo o container com sua propria configuração, primeiramente configure um ou mais arquivos .prop conforme o modelo abaixo

```
schemaspy.db.type=mysql
db.server=172.17.0.1
db.user=root
db.password=secret
```

a partir dai só é preciso chamar o container conforme abaixo indicando o diretorio onde se encontram os arquivos .prop

`docker run -it --rm -p 80:80 -v $PWD/dbconf:/dbconf --name databasediagrams krismorte/databasediagrams:2.0`

agora aproveite seus bancos de dados bem documentados http://localhost/

Ao ver o schemaspy a primeira vez quis muito contribuir ao projeto mas ainda não tive tempo para isso. Minha solução também não ficou perfeita pois trata-se apenas da coordenação da execução do schemaspy então tem muita duplicação de javascript. Quem sabe quando eu finalmente contribuir no projeto minha solução se tornará mais rapida e menor, mas por enquanto faz o trabalho bem feito.