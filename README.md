<img align="right" src="https://raw.githubusercontent.com/startxfr/docker-images/master/travis/logo-small.svg?sanitize=true">

# docker-images-example-postgres


Example of an advanced SQL database using the startx s2i builder [startx/sv-postgres](https://hub.docker.com/r/startx/sv-postgres). 
For more information on how to use this image, **[read startx postgres image guideline](https://github.com/startxfr/docker-images/blob/master/Services/postgres/README.md)**.

## Running this example in OKD (aka Openshift)

### Create a sample application

```bash
# Create a openshift project
oc new-project startx-example-postgres
# start a new application (build and run)
oc process -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/postgres/openshift-template-build.yml -p APP_NAME=myapp | oc create -f -
# Watch when resources are available
sleep 30 && oc get all
```

### Create a personalized application

- **Initialize** a project
  ```bash
  export MYAPP=myapp
  oc new-project ${MYAPP}
  ```
- **Add template** to the project service catalog
  ```bash
  oc create -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/postgres/openshift-template-build.yml -n startx-example-postgres
  ```
- **Generate** your current application definition
  ```bash
  export MYVERSION=0.1
  oc process -n startx-example-postgres -f startx-postgres-build-template \
      -p APP_NAME=v${MYVERSION} \
      -p APP_STAGE=example \
      -p BUILDER_TAG=latest \
      -p SOURCE_GIT=https://github.com/startxfr/docker-images-example-postgres.git \
      -p SOURCE_BRANCH=master \
      -p PGSQL_ROOT_PASSWORD=cbuser \
      -p PGSQL_USER=dbuser \
      -p PGSQL_PASSWORD=dbuser_pwd \
      -p PGSQL_DATABASE=example \
      -p MEMORY_LIMIT=256Mi \
  > ./${MYAPP}.definitions.yml
  ```
- **Review** your resources definition stored in `./${MYAPP}.definitions.yml`
- **build and run** your application
  ```bash
  oc create -f ./${MYAPP}.definitions.yml -n startx-example-postgres
  sleep 15 && oc get all
  ```
- **Test** your application
  ```bash
  oc describe route -n startx-example-postgres
  pgsql -h localhost -P <service-port>
  ```

## Running this example with source-to-image (aka s2i)

### Create a sample application

```bash
# Build the application
s2i build https://github.com/startxfr/docker-images-example-postgres startx/sv-postgres startx-postgres-sample
# Run the application
docker run --rm -d -p 8777:5432 startx-postgres-sample
# Test the sample application
pgsql -h localhost -P 8777
```

### Create a personalized application

- **Initialize** a project directory
  ```bash
  git clone https://github.com/startxfr/docker-images-example-postgres.git postgres-myapp
  cd postgres-myapp
  rm -rf .git
  ```
- **Develop** and create a database content
  ```bash
  cat << "EOF"
  CREATE TABLE `sample` (
  `id` int(4) unsigned NOT NULL AUTO_INCREMENT,
  `key` varchar(128) NOT NULL DEFAULT '',
  `val` varchar(255) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `key` (`key`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  EOF > schema-example.sql
  ```
- **Build** your current application with startx postgres builder
  ```bash
  s2i build . startx/sv-postgres:latest startx-postgres-myapp
  ```
- **Run** your application and test it
  ```bash
  docker run --rm -d -p 8777:5432 startx-postgres-myapp
  ```
- **Test** your application
  ```bash
  pgsql -h localhost -P 8777
  ```
