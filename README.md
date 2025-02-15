# Site Scanning Engine

## Description
_Note: The project is a work in progress and APIs are unstable._

This repository is for the Site Scanning project. This is the new base scanner repository which uses Headless Chrome, powered by Puppeteer for scanning. 

For more detailed documentation about the Site Scanning program, including **who it's for**, **long-term goals**, and **how you can help** please visit the [Site Scanning Documentation Repository](https://github.com/GSA/site-scanning-documentation).


## Table of Contents

* [Quickstart](#quickstart)
    * [Installation](#Installation)
    * [Dotenv](#Dotenv)
    * [Docker](#Docker)
    * [Start all Apps](#start-all-apps)
    * [Ingest Website List](#ingest-website-list)
    * [Test](#test)

* [Development Documentation](./docs)
    * [Project Layout](./docs/layout.md)
    * [Detailed Development](./docs/development.md)
    * [Architecture](./docs/architecture/README.md)


## Quickstart

Development Requirements:

`git`

`nodejs`

`nvm` (see [.nvmrc](./.nvmrc) for current `node` version.

`docker`

`docker-compose`

Cloud Foundry CLI (aka `cf`)

`redis-cli` (optional)

### Installation

First clone the repository:

```bash
$ git clone https://github.com/GSA/site-scanning-engine/
```


From the project root run:

```bash
nvm use
```
This will install the correct Node version for the project.


```bash
$ npm i
```
This will install all production and development Node dependencies.

### Dotenv
The project uses a dotenv (`.env`) file for local development credentials. Note that this file is not version-controlled and should only be used for local development.

Before starting Docker, create a `.env` file in the project root and add the following values replacing `<add_a_key_here>` with a local passwords that are at least 8 characters long.

_**Note: this is only for local development and has no impact on the Cloud.gov configuration**_
```
# postgres configuration
DATABASE_HOST=localhost
DATABASE_PORT=5432
POSTGRES_USER=postgres
POSTGRES_PASSWORD=<add_a_key_here>

# redis configuration
QUEUE_HOST=localhost
QUEUE_PORT=6379

# cron configuration -- you can set these to any valid CRONTAB string
# by default it runs the scans every 5 minutes and the snapshot every 10 minutes
CORE_SCAN_SCHEDULE=*/5 * * * *
SNAPSHOT_SCHEDULE=*/10 * * * *

# Minio Config -- Minio is an S3 api compliant storage
MINIO_ACCESS_KEY=<add_a_key_here>
MINIO_SECRET_KEY=<add_a_key_here>
AWS_ACCESS_KEY_ID=<add_a_key_here>
AWS_SECRET_ACCESS_KEY=<add_a_key_here>
S3_HOSTNAME=localhost
S3_PORT=9000
S3_BUCKET_NAME=site-scanning-snapshot

# Sets the development environment name to dev
NODE_ENV=dev
```


### Docker
From the project root run:

```
docker-compose up --build -d
```

This will build `(--build)` all of the Docker containers and network interfaces listed in the [docker-compose.yml](docker-compose.yml) file and start them running in the background `(-d)`. 

`docker-compose down` will stop and remove all containers and network interfaces. 

Running `docker-compose up --build -d` will rebuild all of the containers. This is useful if you need to wipe data from the 
database, for instance. 

If you encounter any issues starting the containers with `docker-compose`, specifically related to OOM errors (or Exit 137) 
try upping the resources in your Docker preferences.

### Build and Start all apps

`cd` to the project root and run:
```bash
$ npm run build:all
```
This command will build the apps, which compiles from Typescript to Javascript, doing any minification and optimization in the process. All of the app artifacts end up in [/dist](dist). This is ultimately what gets pushed to Cloud Foundry. 

Note that you can also build apps seperately:

```bash
$ npm run build:api
$ npm run build:producer
$ npm run build:scan-engine
$ npm run build:cli
```

Next, you can start the apps with following command:
```bash
$ npm run start:all
```
The apps are started as follows: first the API starts and then the Site Scanner Consumer and Site Scanner Producer follow after the API becomes available. This is designed so that the API app runs any shared configuration against the database first. 

Note, that you can start the apps individually as follows:

```bash
$ npm run start:api
$ npm run start:producer
$ npm run start:scan-engine
```


### Ingest Website List

The Site Scanner relies on a list of federal domains and metadata about those to domains to operate. 
This list is ingested into the system from a public repository using a the [Ingest Service](libs/ingest).

To run the ingest service do the following: 

```bash
npm run ingest -- --limit 200
```

The limit parameter is optional, but it can be useful to use a smaller subset of the total list for local development.

## Test
From the project root run:

```bash
$ npm run test:unit
```
This runs all unit tests. 

## Deploy

First, log in to Cloud.gov using the CLI and choose the organization and space. 

Then, you can use the `cloudgov-deploy.sh` script to build and deploy the apps. 

You can optionally pass a different manifest file with `cloudgov-deploy.sh manifest-dev.yml`.