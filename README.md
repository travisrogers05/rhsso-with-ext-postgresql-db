# rhsso-with-external-postgresql-db

Example of using an external Postgresql database with the Red Hat Single Sign On (SSO) container for Openshift.

This example does not add a Postgresql JDBC driver as a module into a Red Hat SSO image already provides a version of the Postgresql JDBC driver.  Please be aware that this could change in future versions of the RHSSO container where third party JDBC drivers might not be provided and would need to be installed.  A datasource is created at deploy time that uses the Postgresql JDBC driver.  This example assumes that the Postgresql database is visible to pods via DNS alone.

This repository provides a working reference which includes:

- An `.s2i` directory that includes an `environment` [file](https://github.com/travisrogers05/rhsso-with-ext-postgresql-db/blob/master/.s2i/environment) that sets `CUSTOM_INSTALL_DIRECTORIES=extensions`.  This is used by scripts provided in the Red Hat SSO image to allow for customization to take place at pod deploy time.
- A `configuration` [directory](https://github.com/travisrogers05/rhsso-with-ext-postgresql-db/tree/master/configuration) that contains
  - A `datasources.env` [file](https://github.com/travisrogers05/rhsso-with-ext-postgresql-db/tree/master/configuration/datasources.env) that provides all the specifics for the datasource.  These settings are incorporated into the Red Hat SSO configuration at pod deploy time.  Multiple datasources can be provided, although this example uses only one.  Refer to the [JBoss EAP for Openshift documentation](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html-single/red_hat_jboss_enterprise_application_platform_for_openshift/#S2I-Artifacts) for further details about the expected contents of this file.
- A [template file](https://github.com/travisrogers05/rhsso-with-ext-postgresql-db/blob/master/sso72-https-postgresql-external.json) that is derived from an [example template](https://github.com/jboss-openshift/application-templates/blob/ose-v1.4.13/sso/sso72-https.json) provided by Red Hat.  The template contains the foloowing modifications: 
  - Added a [buildconfig](https://github.com/travisrogers05/rhsso-with-ext-postgresql-db/blob/master/sso72-https-postgresql-external.json#L356-#L412) to allow the inclusion of files from this git repo into the image.
  - Added [parameters](https://github.com/travisrogers05/rhsso-with-ext-postgresql-db/blob/master/sso72-https-postgresql-external.json#L45-#L65) for building from a git repository  
  - Added the `ENV_FILES` [environment variable](https://github.com/travisrogers05/rhsso-with-ext-postgresql-db/blob/master/sso72-https-postgresql-external.json#L528-#L531) to allow for providing datasource settings to a pod at deploy time.


## How it works

An Openshift build process clones this git repo into a build pod that performs a maven build of the example servlet.  The Openshift build process produces a container image to be used in application pods.

When the resulting container image is used to produce an application pod, the pod is configured at deploy time to include datasource settings provided by the `datasources.env` [file](https://github.com/travisrogers05/rhsso-with-ext-postgresql-db/blob/master/configuration/datasources.env).  The JBoss EAP configuration file (`/opt/eap/standalone/configuration/standalone-openshift.xml`) that is provided in the Red Hat SSO for Openshift image is updated to include the KeycloakDS datasource configuration backed by a Postgresql database.

