# Helm Examples for JBoss EAP on OpenShift

This repository contains a set of examples for JBoss EAP that can be deployed on OpenShift.

The examples showcased different capabilities of JBoss EAP.

To install them from OpenShift Web Console, a `ProjectHelmChartRepository` needs to be deployed in OpenShift by following the [instructions](https://jboss-eap-up-and-running.github.io/helm-charts/).

Once this is done, all the examples will be available in the OpenShift Web Console.
From the `Developer` perspective, go to `+Add` > `Helm Chart` and check the `Examples for JBoss EAP` in the `Chart Repositorie` section.

# Fork this repository

If you want to use some of these charts as basis for your own application, you can fork this repository and use the relevant chart under the `charts` directory.

Note that this repository is using GitHub Pages to publish the charts and make them available from a `ProjectHelmChartRepository`.

This is not required to deploy your fork of the charts, instead you can directly use the `helm` command-line tool to install them on OpenShift.

For example, the [eap7-with-postgres](./charts/eap7-with-postgres) example can be deployed with the instructions:

```
cd charts/eap7-with-postgres
helm install --dependency-update eap7-with-postgres .
```

To uninstall this chart, run the command:

```
helm uninstall eap7-with-postgres
```
