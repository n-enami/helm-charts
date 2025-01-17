# JBoss EAP 8.0 + PostgreSQL

This Helm chart packages an application with [JBoss EAP 8.0](https://www.redhat.com/en/technologies/jboss-middleware/application-platform) and PostgreSQL to be deployed on OpenShift.

It deploys a persistent PostgreSQL database as well as a Jakarta EE application running in JBoss EAP 7.4 that connects to the database.

## Helm Chart Structure

This examples use the `eap8` Helm chart to build the application image from the source code of the Jakarta EE application using Source-to-Image (S2I). This dependency is declared in the `Chart.yaml` file.

It also use Kubernetes resources to deploy a PostgreSQL database with a persistent volume.

### Deploy the PostgreSQL database

The Helm Chart defines a `postgresql` DeploymentConfig with associated resources to deploy the  PostgreSQL database.

The `values.yaml` file defines 3 fields to configure the name of the database and the user credentials to connect to the database:

```
database:
  name: sampledb
  user: "my-user"
  password: "my-password"
```

### Build and Deploy the Jakarta EE application

The `eap8` Helm chart is used build the application image and is configured in the `values.yaml` file:

```
eap8:
  build:
    uri: https://github.com/jboss-eap-up-and-running/eap8-with-postgres
    env:
      - name: POSTGRESQL_DRIVER_VERSION
        value: '42.7.3'
```

The Jakarta EE application stored in the Git repository [https://github.com/jboss-eap-up-and-running/eap8-with-postgres](https://github.com/jboss-eap-up-and-running/eap8-with-postgres) will be compiled and deployed in a EAP 8.0 server that is provisioned with the `cloud-server` and `postgresql-datasource` layers.

The `postgresql-datasource` layer contains all the artifacts and configuration need to use a Java DataSource that references a PostgreSQL database.
The `POSTGRESQL_DRIVER_VERSION` environment variable must be specified at `build` time to specifiy the version of the JDBC driver.

When the Jakarta EE application is deployed on OpenShift, it must be configured to connect to the PostgreSQL database that has been installed. This configuration is performed with environment variables:

```
eap8:
  deploy:
    env:
      # Env vars to connect to PostgreSQL DB
      - name: POSTGRESQL_DATABASE
        valueFrom:
          secretKeyRef:
            key: database-name
            name: postgresql
      - name: POSTGRESQL_USER
        valueFrom:
          secretKeyRef:
            key: database-user
            name: postgresql
      - name: POSTGRESQL_PASSWORD
        valueFrom:
          secretKeyRef:
            key: database-password
            name: postgresql
      - name: POSTGRESQL_DATASOURCE
        value: qs-ds
      - name: POSTGRESQL_SERVICE_HOST
        value: postgresql
```

The `POSTGRESQL_DATABASE`, `POSTGRESQL_USER` and `POSTGRESQL_PASSWORD` values are read from the `postgresql` secret that is deployed by this Helm Chart (and their values correspond to those specified in the `values.yaml` file).

The `POSTGRESQL_DATASOURCE` value corresponds to the Datasource name used in the Jakarta application's `persistence.xml` file.

The `POSTGRESQL_SERVICE_HOST` value is the name of the service that exposes the PostgreSQL database to the OpenShift cluster.

## Deploying the example with `helm`

Install the Helm chart defined in [https://github.com/jboss-eap-up-and-running/helm-charts/tree/main/charts/eap8-with-postgresql](https://github.com/jboss-eap-up-and-running/helm-charts/tree/main/charts/eap8-with-postgresql) to install the application into OpenShift.

```
git clone https://github.com/jboss-eap-up-and-running/helm-charts.git
cd charts/eap8-with-postgresql
helm install --dependency-update eap8-with-postgresql .
```

## Running the application

Once the application is up and running, you can create a few entries in the database by running the following POST commands:

```
curl -k -X POST https://$(oc get route eap8-with-postgresql --template='{{ .spec.host }}')/app?value=Hello+World
curl -k -X POST https://$(oc get route eap8-with-postgresql --template='{{ .spec.host }}')/app?value=Bonjour+le+Monde
```

Then we can read the entries we added by running the command:

```
curl -k https://$(oc get route eap8-with-postgresql --template='{{ .spec.host }}')/app
```

This command will display the result: 

```
["Hello World","Bonjour le Monde"]
```

## Delete the application

To delete the application with Helm, run the command:

```
helm uninstall eap8-with-postgresql
```

## Source

The source code of the Jakarta EE application is at [https://github.com/jboss-eap-up-and-running/eap8-with-postgres](https://github.com/jboss-eap-up-and-running/eap8-with-postgres).

The source code of the Helm Chart is at [https://github.com/jboss-eap-up-and-running/helm-charts/tree/main/charts/eap8-with-postgresql](https://github.com/jboss-eap-up-and-running/helm-charts/tree/main/charts/eap8-with-postgresql)

