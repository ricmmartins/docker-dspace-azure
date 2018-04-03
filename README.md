# What is DSpace?

![logo](https://github.com/rmmartins/docker-dspace-azure/raw/master/dspace_logo.png)

[DSpace](https://wiki.duraspace.org/display/DSDOC6x/Introduction) is an open source repository software package typically used for creating open access repositories for scholarly and/or published digital content. While DSpace shares some feature overlap with content management systems and document management systems, the DSpace repository software serves a specific need as a digital archives system, focused on the long-term storage, access and preservation of digital content.

This image is based on official [Ubuntu image](https://hub.docker.com/_/ubuntu/) and use [Tomcat](http://tomcat.apache.org/) to run DSpace as defined in the [installation guide](https://wiki.duraspace.org/display/DSDOC6x/Installing+DSpace).

# Usage

DSpace use [PostgreSQL](http://www.postgresql.org/) as database.

Here, I'll use the Azure Database for PostgreSQL as an external database.

## Postgres as a Service

First, we have to create the Azure Database for PostgreSQL. I'm using [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/) to complete the steps described at [https://docs.microsoft.com/en-us/azure/postgresql/quickstart-create-server-database-azure-cli](https://docs.microsoft.com/en-us/azure/postgresql/quickstart-create-server-database-azure-cli)

### Create the resource group
```
az group create --name myresourcegroup --location eastus
```

### Add the extension

Add the updated Azure Database for PostgreSQL management extension using the following command:

```
az extension add --name rdbms
```

### Create the Postgres as a Service

```
az postgres server create --resource-group myresourcegroup --name mydemoserver  --location eastus --admin-user myadmin --admin-password <server_admin_password> --performance-tier Basic --ssl-enforcement Disabled
```

### Create a firewall rule: 

```
az postgres server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowAllIps --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
```
### Create the database and install a required extension: 

Dspace requires the PostgreSQL 'pgcrypto' extension. So we need to create the database and install this extension. I'll use the [psql](https://www.postgresql.org/docs/9.2/static/app-psql.html) command line to do this.

```
psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
create database dspace; 
\c dspace
create extension pgcrypto;
```


# Build

### Clone this repo

```
git clone https://github.com/rmmartins/docker-dspace-azure.git
```

### Make some adjustments

To build the Dockerfile, you need to ajust the [local.cfg](https://github.com/rmmartins/docker-dspace-azure/blob/master/config/local.cfg#L91) at line 91 to change the servername, user and password by yours.


```
cd docker-dspace-azure/
vim config/local.cfg
```

After make the changes, the line 91 from config/local.cfg must be similar to this:
```
db.url = jdbc:postgresql://dspace2.postgres.database.azure.com:5432/dspace?user=dspaceadmin@dspace2&password=Pass0rd1?wx$&ssl=false
```

### Let's build

```
sudo  docker build -t rmartins/docker-dspace-azure .
```

### And let's run

```
sudo docker run  -p 8080:8080 rmartins/docker-dspace-azure
```

After few seconds DSpace should be accessible from:

 - JSP User Interface: http://localhost:8080/jspui
 - XML User Interface: http://localhost:8080/xmlui
 - OAI-PMH Interface: http://localhost:8080/oai/request?verb=Identify
 - REST: http://localhost:8080/rest

# Running on Azure WebApp for Containers

This is the best part. If you want to run Dspace without need to manage VMs and all efforts involved in a [IaaS](https://azure.microsoft.com/en-us/overview/what-is-iaas/) structure, let me present to you the [Azure WebApp for Containers](https://azure.microsoft.com/en-us/services/app-service/containers/). This is allow you to run DSpace in Docker Containers in [PaaS](https://azure.microsoft.com/en-us/overview/what-is-paas/) structure.


## Preparing the image

First save as new image by finding the container ID (using docker ps) and then committing it to a new image name: 

```
sudo docker commit c16378f943fe rmartins/docker-dspace-azure-webapp
```

## Push the imagem to repository

And finally, push the new image to Docker Hub:

```
sudo docker push rmartins/docker-dspace-azure-webapp
```

You can use also the [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/) to store your images :D

## Setup new Webapp for Containers

Create the Azure Webapp for Containers and point to the docker image at your repository, like this:

![webapp](https://github.com/rmmartins/docker-dspace-azure/raw/master/webapp.png)

And make sure that you have the following variables defined:

![settings](https://github.com/rmmartins/docker-dspace-azure/raw/master/settings.png)

## Job done

In a few minutes, you will able to see the Dspace working properly:

### JSPUI

![jspui](https://github.com/rmmartins/docker-dspace-azure/raw/master/running-jspui.png)

### XMLUI

![xmlui](https://github.com/rmmartins/docker-dspace-azure/raw/master/running-xmlui.png)

### OAI

![oai](https://github.com/rmmartins/docker-dspace-azure/raw/master/running-oai.png)

### REST

![rest](https://github.com/rmmartins/docker-dspace-azure/raw/master/running-rest.png)



# License

All the code contained in this repository, unless explicitly stated, is
licensed under Apache License Version 2.0.

A copy of the license can be found inside the [LICENSE](LICENSE) file.
