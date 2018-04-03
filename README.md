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
sudo  docker build -t rmartins/docker-dspace-ssh .
```

### And let's run

```
sudo docker run  -p 8080:8080 rmartins/docker-dspace-ssh
```






After few seconds DSpace should be accessible from:

 - JSP User Interface: http://localhost:8080/jspui
 - XML User Interface: http://localhost:8080/xmlui
 - OAI-PMH Interface: http://localhost:8080/oai/request?verb=Identify
 - REST: http://localhost:8080/rest


This project is configured as an [automated build in Dockerhub](https://hub.docker.com/r/unixelias/docker-dspace/).

# License

All the code contained in this repository, unless explicitly stated, is
licensed under Apache License Version 2.0.

A copy of the license can be found inside the [LICENSE](LICENSE) file.
