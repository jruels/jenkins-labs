# Static Code Analysis with SonarQube

## Running SonarQube as a Docker Container

We assume that docker image for SonarQube is being pulled from [docker hub](https://hub.docker.com/_/sonarqube/). Now use docker run command with port mapping to run SonarQube docker container.

```
docker run -idt  --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube:5.6.4
```

## Install  Sonarqube Plugin in Jenkins

From

```
Manage Jenkins => Manage Plugins
```

install SonarQube plugin from Available tab.

## Generate Token (For SonarQube version 5.3 or higher)

* From SonarQube UI (http://YourIP:9000), login using following (default) credentials.

```
user: admin
pass: admin
```

* Click on **Administration** Tab. From **Security** drop down menu, select **Users**.

* For Administrator User, click on **Tokens**.

![Token ](images/chap11/sonarqube_tokens-00.jpg)

* From **Generate Tokens** provide a name and click on **Generate**.

![Token ](images/chap11/sonarqube_tokens-02.jpg)

Copy over the Token. This should be used for subsequently configuring sonarqube plugin for jenkins. **Please note, this token is shown only once**.

## Configure Sonarqube Plugin

From "Configure System" scroll to **SonarQube servers** and click on "Add SonarQube".

Select the following option,

```
Enable injection of SonarQube server configuration as build environment variables.
```

Provide the details.

```
Name : Sonarqube
Server URL: http://YourIP:9000
Either Token or User/Pass based on your version of SonarQube.
```

Click on Apply.

![Token ](images/chap11/plugin_config.png)


## Create A Job to run Sonar Analysis

From **New Items** create a new project. Name it as **SonarQube** and select Freestyle Project.

Review Job Configurations

* Provide Description.

* Add Git SCM Repository Configuration.

* From Build Triggers, select **Build After Other Projects are Built** and provide dependency on "build".

* From Build Environment , check  **Prepare SonarQube Scanner environment**. If you do not see this option, you will have to go back to system configurations, go to sonarqube server options and check the box which starts with "Enable injection of SonarQube....."

* From **Build**, select **Invoke top-level Maven targets**. From drop down menu, choose the version of Maven to use. In the Goals, provide following line,

```
$SONAR_MAVEN_GOAL -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=admin -Dsonar.password=admin
```

Save and Build.  If the job is successful, it should show a status similar to below,

![SonarQube job status](images/chap11/status.png)


## Adding Sonarqube_Build_Breaker Plugin

* Login to the sonarqube container.

```
docker exec -it sonarqube /bin/bash
```

* Execute the following commands to install plugin and exit the container.

```
cd extensions/plugins/
wget -c https://github.com/SonarQubeCommunity/sonar-build-breaker/releases/download/2.1/sonar-build-breaker-plugin-2.1.jar
exit
```

* Restart the container.

```
docker restart sonarqube
```

## Customizing Quality Gate

* Login as admin in sonarqube console and create a new `Quality Gates` policy, name it as `strict`.

* Choose "Lines of Code" metric and set appropriate warning and error values(like showed in the picture).

  ![Quality Gate](images/chap11/new_sonar_rule.png)

* Once the Build falls under the criteria it will fail with the help of build-breaker plugin that we installed.

  ![Quality Gate](images/chap11/failed_sonar_build.png)

* Thus the build is marked as failure and post build action will not be triggered.

  ![Jenkins_sonar_fail](images/chap11/3.png)

----
:point_left:[**Prev** Chapter 10: Creating Test Job](100_creating_test_job.md)

:point_right: [**Next** Chapter 12: Creating Package  Job](120_create_package_job.md)
