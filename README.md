# rhsso-with-external-postgresql-db

Example of using an external Postgresql database with the Red Hat Single Sign On (SSO) container for Openshift.

This example adds an Oracle JDBC driver as a module into a Red Hat SSO image during an Openshift based source to image (s2i) build.  A datasource is created at deploy time that uses the Oracle JDBC driver.  This example assumes that the Oracle database is visible to pods via DNS alone.

**NOTE:** The Oracle JDBC driver is not provided with this example.  [Download the JDBC driver.](http://www.oracle.com/technetwork/database/application-development/jdbc/downloads)

This repository provides a working reference which includes:

- An `.s2i` directory that includes an `environment` [file](https://github.com/travisrogers05/eap-oracle-db/blob/master/.s2i/environment) that sets `CUSTOM_INSTALL_DIRECTORIES=extensions`.  This is used by scripts provided in the Red Hat SSO image to allow for customization to take place at pod deploy time.
- An `extensions` [directory](https://github.com/travisrogers05/eap-oracle-db/tree/master/extensions) that contains 
  - The necessary [module directory structure and module.xml](https://github.com/travisrogers05/eap-oracle-db/tree/master/extensions/modules/com/oracle/main) file for the Oracle JDBC driver, as well as the Oracle JDBC driver file.  The Oracle JDBC driver must be [downloaded from Oracle](http://www.oracle.com/technetwork/database/features/jdbc/jdbc-ucp-122-3110062.html).
  - A `drivers.env` [file](https://github.com/travisrogers05/eap-oracle-db/blob/master/extensions/drivers.env) that contains driver specific details.  This example includes one driver, but multiple drivers can be included.  Refer to the [JBoss EAP for Openshift documentation](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html-single/red_hat_jboss_enterprise_application_platform_for_openshift/#S2I-Artifacts) for further details about the expected contents of this file.
  - An `install.sh` [file](https://github.com/travisrogers05/eap-oracle-db/blob/master/extensions/install.sh) that is executed during the Openshift s2i build process.  This script takes care of installing the Oracle JDBC driver as a module into the Red Hat SSO image.  (adding files to the image and updating the standalone-openshift.xml to include the driver config)
- A `configuration` [directory](https://github.com/travisrogers05/eap-oracle-db/blob/master/configuration) that contains
  - A `datasources.env` [file](https://github.com/travisrogers05/eap-oracle-db/blob/master/configuration/datasources.env) that provides all the specifics for the datasource.  These settings are incorporated into the Red Hat SSO configuration at pod deploy time.  Multiple datasources can be provided, although this example uses only one.  Refer to the [JBoss EAP for Openshift documentation](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html-single/red_hat_jboss_enterprise_application_platform_for_openshift/#S2I-Artifacts) for further details about the expected contents of this file.
- A [template file](https://github.com/travisrogers05/rhsso-with-oracle-db/blob/master/sso72-https-oracle-external.json) that is derived from an [example template](https://github.com/jboss-openshift/application-templates/blob/ose-v1.4.13/sso/sso72-https.json) provided by Red Hat.  The template contains the foloowing modifications: 
  - Added a [buildconfig](https://github.com/travisrogers05/rhsso-with-oracle-db/blob/master/sso72-https-oracle-external.json#L356-#L413) to allow the inclusion of files from this git repo into the image.
  - Added [parameters](https://github.com/travisrogers05/rhsso-with-oracle-db/blob/master/sso72-https-oracle-external.json#L45-#L65) for building from a git repository  
  - Added the `ENV_FILES` environment variable to allow for providing datasource settings to a pod at deploy time.


## How it works

An Openshift build process clones this git repo into a build pod that performs a maven build of the example servlet.  The example servlet artifact and Oracle JDBC driver are copied into the image during the build.  The JBoss EAP configuration file (`/opt/eap/standalone/configuration/standalone-openshift.xml`) that is provided in the Red Hat SSO for Openshift image is updated to include the Oracle JDBC driver configuration.  The Openshift build process produces a container image to be used in application pods.

When the resulting container image is used to produce an application pod, the pod is configured at deploy time to include datasource settings provided by the `datasources.env` [file](https://github.com/travisrogers05/eap-oracle-db/blob/master/configuration/datasources.env).

