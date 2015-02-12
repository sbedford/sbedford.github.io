---
layout: post
title: SSRS Deployment Automation with Octopus Deploy
byline: How we use Octopus Deploy to automate the release of our SQL Server Reporting Services solution
---

We use SQL Server Reporting Services to provide real-time reporting to the our clients business units spread across the country.  Each of these business units have their own data warehouse which the reports need to query - i.e. the Brisbane BU has its own database adn the Sydney BU its own database.

Anyone who has ever done any SSRS development knows that deploying these reports manually is a real pain in the backside.  Nothing is more painful than trying to upload a large number of files through the clunky SSRS web interface, though manually executing database scripts across multiple databases is pretty ordinary too.

The first solution which people usually move to is automating the deployment through Powershell scripts.  

This was the first step we took, however having to manually run the scripts along with ensuring that the databases were updated with the latest versions of the sprocs and functions was too tedious over time.

Complete deployment automation was the goal - we wanted a totally hands free process which we could reliably and repeatably run in all environments and [Octopus Deploy](http://www.octopusdeploy.com) allowed us to achieve this.

### Packaging

Octopus Deploy requires a nuget package which contains the artefacts to be deployed.  The contents of this nuget package is defined through a nuspec file in the repository

```xml
<?xml version="1.0"?>
<package xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">
  <metadata>
    <id>Reports</id>
    <title>Report PAckage</title>
    <version>1.0.0</version>
    <authors>Ajilon</authors>
    <owners>Ajilon</owners>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>Contains all SQL Server Reporting Services reports and scripts required to create a fully functional environment</description>
  </metadata>
  <files>
    <file src="*.rdl" target="Reports" />
    <file src="..\sql\*.sql" target="Sql" />
    <file src="..\sql\Scripts\*.sql" target="Sql\Scripts" />
    <file src="..\sql\Scripts\Dependencies\*.sql" target="Sql\Scripts\Dependencies" />
    <file src="*.ps1" target="" />
    <file src="..\*.ps1" target="" />
  </files>
</package>
```

Using our existing TeamCity installation, I created a new build configuration to retrieve the SSRS project from GitHub and build the nuget package according to the nuspec using nuget.exe (phew)

![TeamCity Build Configuration](/images/2015-02-12-teamcity-config.png "TeamCity Build Configuration")

### Deployment

The deployment process leverages two powershell scripts invoked from the default.ps1 Octopus script.  The process is illustrated below  and requires a number of configuration variables which are supplied by Octopus depending on the environment which is being deployed to - neat!

![Octopus Deployment Process](/images/2015-02-12-octopus-process.png "Octopus Deployment Process")

The process relies on a naming convention for identifying the data sources as different actions are required for the business unit databases as opposed to a centralised database in use by our core application.

Updating the datasources is required for two reasons: 

1. Credentials in an embeeded data source are not saved when uploading an RDL - also, we're not using the same credentials in development and production environments are we?
1. The link between a Report and a Shared Datasource must be re-established before the report can be run.

* The _dsDynamic_ data source is an expression-based data source which uses a parameter passed into the report to lookup the connection from Oracles tnsnames.ora
* The _dsTas_ data source is a shared data source which is expected to exist in a configurable directory on the reporting server.  This is the only manual step required in the installation and is required as the link between the Report and the SD