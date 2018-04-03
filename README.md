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

By default the database schema is created with the name `dspace` for a user `dspace` and password `dspace`, but it's possible to override this default settings :

docker run -d --link dspace_db:postgres \
        -e POSTGRES_SCHEMA=my_dspace \
        -e POSTGRES_USER=my_user \
        -e POSTGRES_PASSWORD=my_password \
        -p 8080:8080 unixelias/docker-dspace
```

We might also used the Docker compose project in the `sample` directory.

## External database  

When you use an external Postgres, you have to set some environment variables :
  - `POSTGRES_DB_HOST` (required): The server host name or ip(`postgres` by default).
  - `POSTGRES_DB_PORT` (optional): The server port (`5432` by default)
  - `POSTGRES_SCHEMA` (optional): The database schema (`dspace` by default)
  - `POSTGRES_USER` (optional): The user used by DSpace (`dspace` by default)
  - `POSTGRES_PASSWORD` (optional): The password of the user used by DSpace (`dspace` by default)
  - `POSTGRES_ADMIN_USER` (optional): The admin user creating the Database and the user (`postgres` by default)
  - `POSTGRES_ADMIN_PASSWORD` (optional): The password of the admin user


```
docker run -d  \
        -e POSTGRES_DB_HOST=my_host \
        -e POSTGRES_ADMIN_USER=my_admin \
        -e POSTGRES_ADMIN_PASSWORD=my_admin_password \
        -e POSTGRES_SCHEMA=my_dspace \
        -e POSTGRES_USER=my_user \
        -e POSTGRES_PASSWORD=my_password \
        -p 8080:8080 unixelias/docker-dspace
```


After few seconds DSpace should be accessible from:

 - JSP User Interface: http://localhost:8080/jspui
 - XML User Interface: http://localhost:8080/xmlui
 - OAI-PMH Interface: http://localhost:8080/oai/request?verb=Identify
 - REST: http://localhost:8080/rest

Note: The security constraint to tunnel request with SSL on the `/rest` endpoint has been removed, but it's very important to securize this endpoint in production through [Nginx](https://github.com/1science/docker-nginx) for example.

## Configure webapps installed


#### This image removes oai, sword, swordv2 an xmlui by default ***

DSpace consumed a lot of memory, and sometimes we don't really need all the DSpace webapps. So iy's possible to set an environment variables to control the webapps installed :

```
docker run -d --link dspace_db:postgres \
        -e DSPACE_WEBAPPS="jspui xmlui rest" \
        -p 8080:8080 unixelias/docker-dspace
```

The command above only installed the webapps `jspui` `xmlui` and `rest`.

# Build

This project is configured as an [automated build in Dockerhub](https://hub.docker.com/r/unixelias/docker-dspace/).

# License

All the code contained in this repository, unless explicitly stated, is
licensed under Apache License Version 2.0.

A copy of the license can be found inside the [LICENSE](LICENSE) file.
