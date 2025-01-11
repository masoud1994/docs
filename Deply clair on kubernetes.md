# Deply clair on kubernetes

* https://quay.github.io/clair/whatis.html
* https://medium.com/paloit/container-image-scanning-with-coreos-clair-part-1-17152d6a8421
* https://medium.com/paloit/coreos-clair-part-2-installation-integration-558ec664cece#:~:text=To%20deploy%20Clair%20in%20Kubernetes,in%20Kubernetes%20as%20a%20deployment.&text=%23Create%20Clair%20deployment%2C%20this%20will,up%20Postgres%20and%20Clair%20pods.&text=As%20seen%20here%2C%20Clair%20is,0.3%3A6061%20and%2010.20.

* https://www.postgresql.org/docs/current/auth-trust.html

## Install postgre
```
docker run -d -e "POSTGRES_HOST_AUTH_METHOD=trust" -p 5432:5432 postgres:9.6
```

## Architecture
Clair v4 utilizes the [ClairCore](https://quay.github.io/claircore/) library as its engine for examining contents and reporting vulnerabilities. At a high level you can consider Clair a service wrapper to the functionality provided in the ClairCore library.

## How Clair Works
Clair's analysis is broken into three distinct parts.

### Indexing
Indexing starts with submitting a Manifest to Clair. On receipt, Clair will fetch layers, scan their contents, and return an intermediate representation called an IndexReport.

Manifests are Clair's representation of a container image. Clair leverages the fact that OCI Manifests and Layers are content-addressed to reduce duplicated work.

Once a Manifest is indexed, the IndexReport is persisted for later retrieval.

### Matching
Matching is taking an IndexReport and correlating vulnerabilities affecting the manifest the report represents.

Clair is continually ingesting new security data and a request to the matcher will always provide you with the most up to date vulnerability analysis of an IndexReport.

How we implement indexing and matching in detail is covered in ClairCore's documentation.

### Notifications
Clair implements a notification service.

When new vulnerabilities are discovered, the notifier service will determine if these vulnerabilities affect any indexed Manifests. The notifier will then take action according to its configuration.

## Getting Started With Clair
### Releases
All of the source code needed to build clair is packaged as an archive and attached to the release. Releases are tracked at the [github releases](https://github.com/quay/clair/releases).

The release artifacts also include the clairctl command line tool.

### Official Containers
Clair is officially packaged and released as a container at [quay.io/projectquay/clair](https://quay.io/repository/projectquay/clair). The latest tag tracks the git development branch, and version tags are built from the corresponding release.

## Modes
Clair can run in several modes. Indexer, matcher, notifier or combo mode. In combo mode, everything runs in a single OS process.

If you are just starting with Clair you will most likely want to start with combo mode and venture out to a distributed deployment once acquainted.

This how-to will demonstrate combo mode and introduce some further reading on a distributed deployment.

### Postgres
Clair uses PostgreSQL for its data persistence. Migrations are supported so you should only need to point Clair to a fresh database and have it do the setup for you.

We will assume you have setup a postgres database and it's reachable with the following connection string: host=clair-db port=5432 user=clair dbname=clair sslmode=disable. Adjust for your environment accordingly.

### Starting Clair In Combo Mode
At first start clair-db and use postgresql.

A basic config for combo mode can be found [here](https://github.com/quay/clair/blob/main/config.yaml.sample). Make sure to edit this config with your database settings and set "migrations" to true for all mode stanzas. In this basic combo mode, all "connstring" fields should point to the same database and any *_addr fields are simply ignored. For more details see the config reference and deployment models


```
git clone https://github.com/quay/clair.git
```