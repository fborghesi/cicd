# CICD Tool Setup

This project will bring up a set of containers running the tools needed to do CICD on your software projects. These tools are:

* Gogs
* Jenkins
* Nexus


## Starting the services

All you need to do is bring up the containers using docker compose:


```bash
docker compose up -d --build
```

## Stopping the services

Just stop the containers using docker compose:

```bash
docker compose down
```

## Name resolution
You will need to make the following names resolvable by adding this line to your /etc/hosts file:

```bash
127.0.0.1       localhost       gogs.localdev.net jenkins.localdev.net nexus.localdev.net 
```

Keep in mind you may already have an entry for 127.0.0.1 in your hosts file, if that was the case you'll want to merge these names with the ones listed above.

## Setting up Gogs

Gogs can be accessed on http://gogs.localdev.net:23000 (you may change the mapped port on the docker-compose.yml). First time you access that page you'll need to configure:

* **Database**: SQLite3 is the simplest option if you don't have an external one, it will offer reasonable performance and full functionality for local hosted projects. You may go with other options such as MySQL/PGSql, but it's up to you to set these external services.
* **Application General Settings**: You'll want to make the following changes:
	* **Domain**: localdev.net
	* **Application URL**: http://gogs.localdev.net:23000 (or some other port if you changed it)

These are the minimal settings, feel free to make any other changes that suit your needs. Now click on **Install Gogs** and follow the **Need an account?** link to create a new account.

For ssh access, port 22 has been mapped to 20022.


## Setting up Nexus
Nexus' website can be accessed at http://nexus.localdev.net:8081/. You'll want to sign in using the admin account, the password can be obtained with the command:

```bash
nexus-data/admin.password
```
Click on the *gear* icon to access the **Server administration and configuration** section. There you'll want to create a user for Jenkins to be able to push images into the repo, so:
   * Click on the **Create user** button.
   * Fill in all the fields.
   * Make sure the user is in the *nx-admin* group so it gets write access to the repo.
	
***Important: Write the id and password in a secure place, you'll need them later when creating the credentials in Jenkins.***

Now go to the **Security** section on the left, click on **Realms** and make sure the **Docker Bearer Token Realm** is active.

Finally go to the **Repository** section, click on **Repositories** and create the following ones:
	* maven2 (hosted)
		* **Name**: maven
	* docker (hosted):
		* **Name**: docker
		* **HTTP**: enable on port 5000
		* **Deployment policy**: make sure Allow Redeploy is selected.
	* helm (hosted):
		* **Name**: helm


Feel free to create any other repos you need.


## Setting up Jenkins

To access Jenkins, just open the following link in your browser http://jenkins.localdev.net:28080. You'll be asked for a password which you can get with the following command:

```
docker compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Then **Install suggested plugins** (or **Select plugins to install** if you have specific needs) and **Create First Admin User**. These steps should be trivial.

In the **Instance Configuration** section, please enter http://jenkins.localdev.net:28080 for the Jenkins URL. That's it.

Now it's time to set Jenkins up so click on the Manage Jenkins menu option. Visit the sections below to configure Jenkins.
### Global Tool Configuration
In this section you'll set up the tools you'll need for building your jobs. You'll want to:

* Configure a JDK according to your needs.
* Set up Maven and/or Gradle.
* Set up other tools if needed.
	
### Manage Nodes and Clouds
Your Jenkins image already contains docker-cli, and the Jenkins account is able to invoke docker for image building and pushing. We'll add the *docker* label to the master node so it can be used as a docker agent in your pipelines.

* Select the *master* node
* Select **Configure**
* Add "docker" as a label in the *Labels* field
	
### Manage Credentials
Credentials will let your pipelines interact with other services.

#### Nexus Credentials

* In the **Stores scoped to *Jenkins*** section, select Jenkins
* Then click on **Global credentials (unrestricted)**
* Then click on **Add Credentials** to add the Nexus credentials that will let you push images and packages to the repo:
    * Select *Username with Password* in the **Kind* field.
    * Enter the *username* and *password* you picked in the **Setting Up Nexus section above**.
    * For the Id field, just enter *nexus*.

#### Jenkins Credentials
* In the **Stores scoped to *Jenkins*** section, select Jenkins
* Then click on **Add Credentials** again to add the Git credentials that will let you download code from Gogs. You can use your own account or you can define a separate account in Gogs for Jenkins to use. The latter is recommended.
    * Select *SSH Username with private key* in the **Kind* field.
    * Fill the username and paste the contents of *~/.ssh/id_rsa* (or the file containing the appropiate private key for the account).
    * You may want to enter the passphrase if one is needed.


# Other things to consider

## Pushing docker images into nexus
For docker to be able to push images into the non-HTTPS registry installed above, you'll need to add the following to your docker settings:

```
{
  "insecure-registries": [
    "nexus.localdev.net:5000",
  ]
}
```
Docker settings can be found on ```/etc/docker/daemon.json``` in moste linux distros, or ```~/.docker/daemon.json``` on MacOS.

You can validate your credentials with ```docker login nexus.localdev.net:5000``` to make sure your changes have been accepted.


## Gogs' webhooks
Setting up webhooks on Gogs is very easy, just pick a project, click on **Settings** and then **Webhooks**. 
For the payload URL you'll use: ```http://jenkins:8080/gogs-webhook/?job=<project_name_here>```
The **Test Delivery** button will help you diagnose connectivity.




 



