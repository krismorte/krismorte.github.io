---
title: "Documenting Databases With Schemaspy"
date: "2020-04-12T18:55:25Z"
tags: ["java","database"]
---

When I think about database's documentation remember me of centralizing tools where everything about databases was born on the DBA's hands, design, to chaneg a columntype was needed open a ticket and thing like that. In a world without migrations and CI\CD makes totally sense give that power to the right guy, huge databases with tons of objects and data somethime attend more than one sysmte at time make us need a [benevolent dictator](https://git-scm.com/book/en/v2/Distributed-Git-Distributed-Workflows).

In this new microsservice's world, CI\CD and migrations all the decision power of design was handled to the application, databases each time got smaller and focus and its own system and free of structures that holds some kind of bussines rules. In this new world is still worth have some kind of centralizing tools to manager databases? Well, at this point I found [SchemaSpy](http://schemaspy.org/) and really believe that the answer is no.

Schamespy is an application build on Java thats scan you entire database and generates a static web site containing all tables, views, functions and relations between them besides creatio script from some. Imagine "to browse" through your databases in a totally visual way included graphics and simple anomaly analysis. Yes it's the guy.

Let's put our hand on it. Will be necessary install graphviz, java and download the last verions from SchemaSpy and the jdbc drive from the database that you want document, that simple. The basic call is by commadn line as showed below

`java -jar schemaspy.jar -t mssql05 -dp C:/sqljdbc4-3.0.jar -db DATABASE -host SERVER -port 1433 -s dbo -u USER -p PASSWORD -o DIRECTORY`

from the command we have to see `-t`  that indicates which databases type, you can check all supported [here](https://github.com/schemaspy/schemaspy/tree/master/src/main/resources/org/schemaspy/types), `-dp` this one indicates where is the jdbc drive or the folder and `-o` indicates the destiny to gereates the static site. We also have `-s` to indicate schema, to sql server is not mantatory but to mysql is and ptretty helpful to oracle.

Until here everything is perfect isn't? Well, not for me, when I found this solution helps me a lot, but shortly thereafter came to my mind that my database server hold more than 40 databases and schemaspy just document one per time. Behold, it came to me an idea to wrapper schemaspy in my own solution to at least documenting a whole server per time and was born [Database Diagrams](https://github.com/krismorte/database-diagrams). 
Being straight forwards is a docker image with nginx to show the sites, a cron job to schedule and a small java application to configure and manager schemaspy executions. How to use? The image is availabe on [docker huh](https://hub.docker.com/repository/docker/krismorte/databasediagrams)

`docker pull krismorte/databasediagrams:2.0`

If you want to test first i recommend clone the git repo and run the docker-compose

```
git clone https://github.com/krismorte/database-diagrams.git
cd database-diagrams
docker-compose up
```

Up a container with your won configuration,firsat of all configure one or more .prop files as I'm showing you below

```
schemaspy.db.type=mysql
db.server=172.17.0.1
db.user=root
db.password=secret
```

from there you need to call you container indicating the folder with all .prop files 

`docker run -it --rm -p 80:80 -v $PWD/dbconf:/dbconf --name databasediagrams krismorte/databasediagrams:2.0`

now enjoy your documented databases documenting http://localhost/

At first sight on schemaspy I wanna contribute to the project but I didn't get the time yet. My solution wasn't perfect because just coordinate the schemaspy executions so there's a lot of javascript duplications. Who knows when I finally will contribute to the project making my solution faster and smaller but for now it's get the job done.