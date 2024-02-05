**Needeed to follow this setup : Docker daemon**

Otherwise it is host here : http://efrei-file-sharing.thibaulthenrion.com/

*The following prompts are bash prompts*


## Step 1 : Docker network creation 

``` bash
docker network create networkAdvProg
```

## Step 2 : Launching the authentication microservice's database

- Launch a PostgreSQL database with the following command:

```bash
docker run -d --name authpostgres \
    -e POSTGRES_PASSWORD=strongPassword \
    -e POSTGRES_USER=admin_authenticator \
    -e POSTGRES_DB=authenticator \
    -p 27018:5432 \
    --network=networkAdvProg \
    postgres
```

## Step 3 : Launching the authentication microservice

- Start the authentication microservice with the following command:

```bash
docker run -d --name authenticator \
    -e SPRING_DATASOURCE_URL=jdbc:postgresql://authpostgres:5432/authenticator \
    -e SPRING_DATASOURCE_USER=admin_authenticator \
    -e SPRING_DATASOURCE_PASSWORD=strongPassword \
    -e INITIAL_PASSWORD=adminadmin \
    -p 27019:9090 \
    --network=networkAdvProg \
    thibaulthen/authenticator:1.8
```

* You now have three different accounts usable with the same password corresponding to INITIAL_PASSWORD (adminadmin by default) in the precedent docker run commmand, here are their usernames : 
    * Student-demo
    * Tutor-demo
    * *Admin-demo*

**You will have to use one of the two firsts account to log-in and test the app**

/!\ Be careful, *Admin-demo*'s only purpose is to manage the authentification service, **DO NOT** use it to login and interact with the document microservice.

## Step 4 : Launching the document management microservice database

- Launch a PostgreSQL database with the following command:

```bash
docker run -d --name docpostgres \
    -e POSTGRES_PASSWORD=anotherPassword \
    -e POSTGRES_USER=admin_document \
    -e POSTGRES_DB=document \
    -e PGPORT=5433 \
    -p 27020:5433 \
    --network=networkAdvProg \
    postgres
```

## Step 5 : Launching the document management microservice

- Start the document management microservice with the following command:

```bash
docker run -d --name documentService \
    -e SPRING_DATASOURCE_URL=jdbc:postgresql://docpostgres:5433/document \
    -e SPRING_DATASOURCE_USER=admin_document \
    -e SPRING_DATASOURCE_PASSWORD=anotherPassword \
    -e AUTH_ISSUER_URL=http://authenticator:9090/ \
    -p 27021:9091 \
    --network=networkAdvProg \
    thibaulthen/document-api:1.4
```

## Step 6 : Launching the Front-End

- Start the react frontend with the following command:

```bash
docker run -d --name advprog-front \
    -e REACT_APP_DOCUMENT_API_URL=http://localhost:27021 \
    -e REACT_APP_AUTH_API_URL=http://localhost:27019/api \
    -p 3000:3000 \
    thibaulthen/front-document-api:1.0
```

**You can now access our application by going to http://localhost:3000 in your browser.**

Then, use the credentials (Tutor or Student) from step 3 to connect.
