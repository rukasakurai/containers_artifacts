# success criteria

- Each member of your team must show your coach a locally running Points of Interest (POI) container connected to a running SQL Server container. Verify that your POI container is serving content via HTTP commands. Explain your setup to your coach and how it could be used for development and testing.

- Your team must have built images for all the TripInsights components and pushed them to the team’s ACR. Share your understanding of how each of the images were built and pushed to the registry with your coach.

# Dockerfiles

POI (.Net Core) - Dockerfile_3
Trips (Go) - Dockerfile_4
User (NodeJS) - Dockerfile_2
User (Java) - Dockerfile_0
TripViewer (.Net Core) - Dockerfile_1

docker run --network <networkname> -e SQLFQDN=<servername> -e SQLUSER=sqladminyGo2491 -e SQLPASS=<password> -e SQLDB=mydrivingDB <your-registry>.azurecr.io/data-load:1.0

# custom network ?

docker network create local-net

# Run SQL server

docker run \
 --network local-net \
 -p 1433:1433 \
 -e "ACCEPT_EULA=Y" \
 -e "SA_PASSWORD=hL2iz6Kd3" \
 --name sql1 \
 -h sql1 \
 mcr.microsoft.com/mssql/server:2017-latest

docker exec -it sql1 "bash"

/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "hL2iz6Kd3"

https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-bash

## create datanase

CREATE DATABASE mydrivingDB
SELECT Name from sys.Databases
Go

## create user

CREATE LOGIN sqladminyGo2491 WITH PASSWORD = 'hL2iz6Kd3';

CREATE USER sqladminyGo2491 FOR LOGIN sqladminyGo2491;

ALTER SERVER ROLE [sysadmin] ADD MEMBER [sqladminyGo2491];

# run dataload container

docker run \
 --network local-net \
 -e SQLFQDN=sql1 \
 -e SQLUSER=sqladminyGo2491 \
 -e SQLPASS=hL2iz6Kd3 \
 -e SQLDB=mydrivingDB \
 registryygo2491.azurecr.io/dataload:1.0

SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES;
GO

# build POI container

docker build -t registryygo2491.azurecr.io/poi . -f ../../dockerfiles/Dockerfile_3

# run POI container

--network host \

docker run \
 --network local-net \
 -p 8081:8081 \
 -e "SQL_USER=sqladminyGo2491" \
 -e "SQL_PASSWORD=hL2iz6Kd3" \
 -e "SQL_SERVER=sql1" \
 -e "SQL_DBNAME=mydrivingDB" \
 -e "WEB_PORT=8081" \
 -e "WEB_SERVER_BASE_URI=http://0.0.0.0" \
 -e "CONFIG_FILES_PATH=/secrets" \
 -e "ASPNETCORE_ENVIRONMENT=Local" \
 --name poi \
 -h poi \
registryygo2491.azurecr.io/poi:latest

docker run --rm -d --network host --name my_nginx nginx

curl -i -X GET 'http://localhost:8081/api/poi'

# build all containers

POI (.Net Core) - Dockerfile_3 - NEEDS DB CONNECT
Trips (Go) - Dockerfile_4 - NEEDS DB CONNECT
User (NodeJS) - Dockerfile_2
User (Java) - Dockerfile_0 -- NEEDS DB CONNECT
TripViewer (.Net Core) - Dockerfile_1

cd /Volumes/BACKUP/Dropbox/AZUREsample/2021-OH-container/collab/containers_artifacts/src/poi
docker build -t registryygo2491.azurecr.io/poi . -f ../../dockerfiles/Dockerfile_3

cd /Volumes/BACKUP/Dropbox/AZUREsample/2021-OH-container/collab/containers_artifacts/src/trips
docker build -t registryygo2491.azurecr.io/trips . -f ../../dockerfiles/Dockerfile_4

cd /Volumes/BACKUP/Dropbox/AZUREsample/2021-OH-container/collab/containers_artifacts/src/user-java
docker build -t registryygo2491.azurecr.io/user-java . -f ../../dockerfiles/Dockerfile_0

cd /Volumes/BACKUP/Dropbox/AZUREsample/2021-OH-container/collab/containers_artifacts/src/userprofile
docker build -t registryygo2491.azurecr.io/userprofile . -f ../../dockerfiles/Dockerfile_2

cd /Volumes/BACKUP/Dropbox/AZUREsample/2021-OH-container/collab/containers_artifacts/src/tripviewer
docker build -t registryygo2491.azurecr.io/tripviewer . -f ../../dockerfiles/Dockerfile_1
