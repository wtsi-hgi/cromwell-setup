# cromwell-setup
Our test setup for Cromwell on (and off) Google Cloud

# Cromwell documentation
Cromwell is the scientific workflow engine from the Broad Institute https://github.com/broadinstitute/cromwell

It can run with or without an external database and locally or cloud based, the config file determines how it runs.

We have three config files for testing, completely local, local run with database backend, and Google Cloud run with database backend. The tests using the syndip data were run on an open stack instance.

The documentation to set it up is

http://cromwell.readthedocs.io/en/stable/Configuring/#database

http://cromwell.readthedocs.io/en/stable/tutorials/PipelinesApi101/

http://cromwell.readthedocs.io/en/stable/tutorials/PersistentServer/

# Database

We have a Cromwell MySQL database set up and monitored by the Database group at Sanger. We have two logins, one admin and one that allows read/write but not changing tables. It's safer to run with the read write only but upgrades of Cromwell might make table changes, and you need to use the admin login at least when starting for the first time or upgrading as it makes it's own tables.

They can make us a more heavy duty database if we decided to use this beyond the test runs. 

As a temporary solution we have a test/development database to explore moving the database component out of OpenStack.  Set up with daily backups, but they don't guarantee to provide point-in-time recovery or failover at this stage.  If we then decide to go forward with a MySQL database they can create a full production environment with on-site and off-site replication slaves in addition to daily backups.  This is their standard production configuration, providing failover to a replication slave and point-in-time recovery if needed.'

# Specific advice on settings

To run Cromwell in production, it gets 16Gs of memory. 

For 8Gs "java -Xmx8g -Xms8g  -Dconfig.file=google-sql.conf -jar cromwell-33.1.jar server"

Another thing that helps is limiting the number of jobs that run simultaneously, as that's what ends up spiking Cromwell's memory. So this is another line you can add to your config:

backend.providers.JES.concurrent-job-limit = 500

# Security

Cromwell has no security itself, https://cromwell.readthedocs.io/en/stable/Security/

For the test run I used a proxy nginx server requiring a login and forwarding to Cromwell. The openstack url for cromwell is also only viewable internally. I shut down the port to all except the proxy. For serious use we might need to rethink.

# Problems

One run seemed to have crashed in the middle of a run, and the logs were unclear. It seemed it could have been memory related. After restarting the server with more memory it didn't recur, but the test workloads were not heavy.

At the moment Cromwell can run CWL workflows locally but they are incompatible with Google's pipelines API so jobs sent to Google Cloud need to be WDL. This should be changing with the next version of the pipelines API. 

# Commands

Start the server:
"java -Xmx8g -Xms8g  -Dconfig.file=test.conf -jar cromwell-33.1.jar server" (or similar, with the version, memory and config file you want to use)

Submit a workflow: 
 curl -X POST "<ip address>/api/workflows/v1" -H  "accept: application/json" -H  "Content-Type: multipart/form-data" -F "workflowSource=@PairedEndSingleSampleWf.gatk4.0.wdl;type=" -F "workflowInputs=@PairedEndSingleSampleWf.hg38.inputs.json;type=application/json" -F "workflowOptions=@PairedEndSingleSampleWf.gatk4.0.options.json;type=application/json" -F "workflowType=WDL" -F "workflowTypeVersion=draft-2"
  
 API calls listed here https://cromwell.readthedocs.io/en/develop/api/RESTAPI/
 
 The server has a 'Swagger' page that allows you to submit commands and select options from the page rather than commandline. This returns you the uuid of the run and also gives the command that is submitted so it's helpful to get started.




