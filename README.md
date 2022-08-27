# Liquibase with MS SQL Server

#### Run your own instance of MS SQL Server in a Docker container and then use a containarized Liquibase instance to manage and compare tables from different databases. 

This project has 3 sections:

1. **Getting started** - which is in this file.
2. **[Demo-1](./DEMO-1.md)** - a step-by-step tutorial covering the basics of using Liquibase on MSSQL. 
3. **[Demo-2](./DEMO-2.md)** - a more advanced database comparison with Liquibase


> Liquibase recommends using Docker to avoid all the installation complications and dependencies
> coming with it.

<br/>

## Getting Started

---
### 1. Clone this project
Clone this project to a local folder in your computer.

--- 
### 2. Download and install **[Docker Desktop](https://docs.docker.com/engine/install/)**.

---

### 3. Create a Docker account
If you don't already have one, you must create a free [Docker account](https://hub.docker.com/signup/) to pull images from [Docker Hub](https://hub.docker.com/).

If this seems a bit annoying to you, well, it is. However, every software engineer must know and work with containers
and this project is one great way to get exposed to Docker, even if you never used it before.

---
### 4. Download and install a SQL Server Editor 
One excellent option is **[DBeaver](https://dbeaver.io/)** as it works Mac, Windows, and Linux. You may use any other SQL editor that works with SQL Server.

--- 
### 5. Start SQL Server in a Docker Container
 ```shell
cd <this project folder in your computer>
```

Pull the image from Docker Hub:
```docker
docker compose up
```

This commands reads the `docker-compose.yml` file in the project's root folder and pulls the image defined in this file.
It may take a few minutes to run, as the first time it downloads (pulls) the SQL Server image to start the
DB container.

This task is not running in the background and after a few minutes you should see no errors in the terminal output.

--- 
### 6. Start the Demo
That's it. Leave the terminal with the SQL Server container running and go to the [DEMO-1.md](DEMO-1.md) file and follow the steps there to see how we can use Liquibase with
MS SQL Server.

---
### Optional
If you never used Docker, check the downloaded image:
```Docker
docker images
```
You should see the image information pulled from Docker Hub.

To view the running container(s) in your computer:
```docker
docker ps 
```
Containers are image instances and in this case we have only one container running from the MS SQL Server image.

Look at the ``./docker-compose.yml`` file to see how the MS SQL Server container is setup. This file was created
so we can easily start the container with a simple ``docker compose up`` command, otherwise we'd need to
execute ``docker run ...`` with all the parameters speficied in ``docker-compose.yml``. Docker compose
can do a lot more than that, but for this demo we use it to make it easy to start the SQL Server container.
