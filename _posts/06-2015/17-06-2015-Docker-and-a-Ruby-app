#Ohana meets Docker

##Introduction
Much like the [Ohana Vagrant Virtual Machine](https://github.com/codeforamerica/ohana-api-dev-box) "This project automates the setup of a development environment for working on Ohana API" This project also includes the Ohana-Search app, furthermore That project uses Vagrant and this uses Docker...

##What you'll need
Docker (v.1.3 at time of writing)

##Inside the box
[The code-for-america 'Ohana-Api'](https://github.com/codeforamerica/ohana-api/blob/master/INSTALL.md) relies on a number of software tools before it can be installed. It also requires a PostgreSQL database to be up and running.

Here you will find Dockerfiles to meet these pre-requisites. The PostgreSQL database runs on a container that is linked to another container running the Ohana-api and Ohana-search apps. Check out the roadmap at the bottom of this document for future work. 

##PostgreSQL 
Pull and run the official Postgresql Docker image:
```
docker run --name db -d postgres
```
Enter the running container
```
docker exec -it db bash
```
Set the pgres database to listen on all addresses and allow password-less access
```
echo "listen_addresses='*'" >> /var/lib/postgresql/data/postgresql.conf
echo "host all all 0.0.0.0/0 trust" >> /var/lib/postgresql/data/pg_hba.conf
```
`ctrl+d` to exit container


Restart the container to propogate changes to config
```
docker stop db
docker start db
```

##Ohana-Api & Ohana-Search Dockerfile

Clone this directory
```
git clone https://github.com/gl2748/ohana-dockerfiles.git
```
Build the docker ohana image
```
docker build -t="imaitland/ohana" . 
```
Run the ohana image (thereby making it a 'container'), open the ports 80 and 22 to the host machine (i.e. the wider world), link it to the pgres database container, name it ohana-app  
```
docker run -d -i -p 80:80 -p 2222:22 --link db:db --name=ohana-app imaitland/ohana
```
Enter the running container
```
docker exec -i -t ohana-app bash
```
Run the dbconfig.sh script
```
dbconfig.sh
```
run the ohana bootstrap script from `/home/ohanauser/ohana`
```
script/bootstrap
```
Start the ohana api app (daemon mode) NB. you need to enter this command from the ohana directory at `/home/ohanauser/ohana`
```
rails s -d -p 22
```
Great the ohana-api admin portal is up and running on port `:2222` of your host machine, for example `178.62.65.184:2222`

---

Now configure the ohana search app so that it can find the api. Go to`/home/ohanauser/ohana-search/config/application.yml` and edit the`OHANA_API_ENDPOINT` line accordingly, for example `OHANA_API_ENDPOINT: http://178.62.65.184:2222/api`


Start the ohana search app NB. you need to enter this command from the ohana-search app directory at `/home/ohanauser/ohana-search`
```
rails s -d -p 80
```
`ctrl+d` (exit container)

Check that the two containers are still up and running
```
docker ps
```

That's it, navigate to the IP of the host server.  for example: `178.62.65.184`

##Roadmap

Ideally the ohana-app will be run from a non-root user aka 'ohanauser'.

