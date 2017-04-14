## A Thousand and One Way to deploy OpenShift Application

The hardest thing when starting OpenShift from a developer point of view is not in deploying an application to it. This step can be done very easily. We found indeed that the most challenging part is to find the best way to do it ! Because OpenShift is so adaptable and flexible that it offers _A Thousand and One Way_ to deploy an application. The _best way_ will be the one that best fit a bunch of criteria:
* Your development process: what is your existing ecosystem and flow? (already using Jenkins, Nexus, ...)
* Your developer's maturity regarding Containers and CI/CD: do they want to manage Containers Dockerfile or Pipeline as code ?
* Your Ops or Technical exports involvement: wether they need to customize provided base images or base templates for introducing specificities,
* Your will to build different Service offers allowing multiple workflows adapted to multiple development teams.

Following this tutorial, you will deploy the same application using 6 different methods. Remember that this is _just_ 6 methods that represents main ways of doing thins, there's a number of variations that may be defined using some elements picked from one or another method exposed here.

For deploying the application repeatedly, you will only need an OpenShift project for isolating things out. Methods #6 may also require a `JBoss Developer Studio` on version 10+. We propose you create this project using the identifier `ocp-tasks` because some methods have this id hard-coded.

So startup by creating the project using the following command:
```
oc new-project ocp-tasks --display-name="OpenShift Tasks" --description="Demonstrating many ways to deploy application on OpenShift"
```
and here we go!

### #1 - Use provided JBoss EAP 7 S2I template

This is the most obvious for people starting with OpenShift: you'll use the default template and base image provided by OpenShift without any customization upfront. To illustrate this flow within your `ocp-tasks` project:
* Go to "Add to project" page,
* Pick the `jboss-eap70-openshift:1.4` template,
* Adapt the name of application to `tpl-tasks` or something,
* Refer this Github repository (`http://github.com/lbroudoux/openshift-tasks`) or another internal one,
* Optionally, add some extra environment variables on the BuildConfig for referring Maven component mirror (`MAVEN_MIRROR_URL`),
* Hit the save button and you'll got the application running in few minutes.

The process executed by OpenShift can be described as follow:

![base-template-s2i](https://raw.githubusercontent.com/lbroudoux/openshift-tasks/master/assets/base-template-s2i.png)  

* Giving the template configuration and given URL for source repository, OpenShift realize the checkout of code sources,
* It sees that application if powered by Maven and so launch the Maven build to that it produces compiled Java classes and then deployable binary artifact (here called `openshift-task.war`),
* That later binary artifact is then marged on top of `jboss-eap70` base image to produce a Docker image for our application,
* Image is tored within OpenShift Docker registry and can then trigger a new deployment (because template says that a new image should trigger a new deployment).

### #2 - Bring your own S2I template

This variant consists in providing your own template that will be a customization of OpenShift provided template. Providing your own template allows you to lower the number of parameters the developer will have to fill and fixing hard values. To illustrate this flow within your `ocp-tasks` project:
* Register your app template in OpenShift using `oc create -f https://raw.githubusercontent.com/lbroudoux/openshift-tasks/master/app-template.yaml -n ocp-tasks`,
* Go to "Add to project" page,
* Pick the `openshift-tasks` template,
* Adapt the name of application to `s2i-tasks` or something,
* Refer this Github repository (`http://github.com/lbroudoux/openshift-tasks`) or another internal one,
* Hit the save button and you'll got the application running in few minutes.

The process executed by OpenShift can be described as follow and is basically the same as with standard base template:

![base-template-s2i](https://raw.githubusercontent.com/lbroudoux/openshift-tasks/master/assets/custom-template-s2i.png)  

The main differences you may noticed when creating your application is that the form to complete before deploying is drastically simpler than the default one. Providing a custom template allows you to pre-configure or hard write some parameters so that you're sure that users would not be able to change values or be bothered by choosing the right values.

### #3 - Custom S2I for binary deployment

This variant consists in providing your own template like the previous one but adapting the Source-to-image process by making OpenShift build your application image not from source but from a binary file you will inject. This variant is using the S2I customization capabilities as described here https://docs.openshift.com/container-platform/3.4/dev_guide/builds/build_strategies.html#override-builder-image-scripts. The customization script we used is located into a companion repository [here](http://github.com/lbroudoux/openshift-tasks-bin-deploy) to enforce separation of concerns but it could be located in same repository.

To illustrate this flow within your `ocp-tasks` project, we assume that previous step has been done and template is already registered. The whole process of this variant is fully described here showing the split of responsabilities between Jenkins and OpenShift:

![s2i-binary-deployment](https://raw.githubusercontent.com/lbroudoux/openshift-tasks/master/assets/s2i-binary-deployment.png)  

The trick here is to use a Jenkins instance to host the first part of your build process (like you would normally do in a traditional way without OpenShift). For doing that you may of course use a Jenkins instance deployed into your OpenShift project. You can easily create one using the `jenkins-persistent` template.

In this Jenkins instance, create a new job item (calling it `bin-tasks` for example) and configure it as a regular Jenkins Maven build. In configuration, you may want to checkout this Github repository (`http://github.com/lbroudoux/openshift-tasks`), then to realize a `mvn package` and finally store the generated artifacts using Jenkins built-in store. This last choice is for letting things simple: in a real-world scenario, you would prefer storing the generated artifact into a Nexus or Artifactory repository.

Now the second part is related to OpenShift. Because previous variant has been done, you are now able to:
* Go to "Add to project" page,
* Pick the `openshift-tasks` template,
* Adapt the name of application to `bin-tasks` or something,
* Refer the companion Github repository (`http://github.com/lbroudoux/openshift-tasks-bin-deploy`) or another internal one,
* Hit the save button.

Build and the Deployment may start but you will likely deploy an empty JBoss EAP because you have not supplied any deployable artifact yet. So you may cancel automatically started build and deployment before going further.

You now have a build config in Jenkins responsible for producing the binary and a build config in OpenShift that will be responsible of collecting the binary artifact and building a container image for deployment. Having a closer look at [assemble file](https://github.com/lbroudoux/openshift-tasks-bin-deploy/blob/master/.s2i/bin/assemble), you'll notice that our Build in OpenShift will need further environment variables to be able to retrieve the artifact. Through the console, go to your `bin-tasks` build and add these vairables:
* `WAR_FILE_URL` will be URL to which is published your artifact by Jenkins. Something like http://jenkins-ocp-tasks.example.com/job/bin-tasks/lastSuccessfulBuild/artifact/target/openshift-tasks.war
* `WAR_FILE_USER` is a user that is allowed to get this file,
* `WAR_FILE_PASSWORD` is the Jenkins API token for this user (go to User settings in Jenkins and reveal API Token).

Finally, you may want to link all things together. So that when the `bin-tasks` job in Jenkins ends up successfully, it may trigger the `bin-tasks` build configuration into OpenShift. This can be easily done via a new Jenkins job that will use the OpenShift plugin for Jenkins. Create such a new job called for example `bin-tasks-deploy` (if your Jenkins runs into OpenShift, you may clone de _OpenShift Sample_ job) and configure this job for watching `bin-tasks` successful result and then trigerring a new OpenShift Built: this is a builtin Build action coming with OpenShift plugin for Jenkins.

### #4 - Build pipeline managed out of Source code


### #5 - Build pipeline managed within Source code

* Register your app template in OpenShift using `oc create -f https://raw.githubusercontent.com/lbroudoux/openshift-tasks/master/app-template-jenkinsfile.yaml -n ocp-tasks`,
* Go to "Add to project" page,
* Pick the `openshift-tasks-jenkinsfile` template,
* Adapt the name of application to `jkf-tasks`,
* Refer this Github repository (`http://github.com/lbroudoux/openshift-tasks`) or another internal one,
* Hit the save button.

![jenkinsfile-template](https://raw.githubusercontent.com/lbroudoux/openshift-tasks/master/assets/jenkinsfile-template.png)

### #6 - Directly from IDE



## OpenShift Tasks: JAX-RS, JPA quickstart

### What is it?

The `tasks-rs` quickstart demonstrates how to implement a JAX-RS service that uses JPA 2.0 persistence deployed to Red Hat JBoss Enterprise Application Platform.

The application manages User and Task JPA entities. A user represents an authenticated principal and is associated with zero or more Tasks. Service methods validate that there is an authenticated principal and the first time a principal is seen, a JPA User entity is created to correspond to the principal. JAX-RS annotated methods are provided for associating Tasks with this User and for listing and removing Tasks.

_Note_: This quickstart uses the H2 database included with Red Hat JBoss Enterprise Application Platform 6. It is a lightweight, relational example datasource that is used for examples only. It is not robust or scalable, is not supported, and should NOT be used in a production environment!_

_Note_: This quickstart uses a `*-ds.xml` datasource configuration file for convenience and ease of database configuration. These files are deprecated in JBoss EAP 6.4 and should not be used in a production environment. Instead, you should configure the datasource using the Management CLI or Management Console. Datasource configuration is documented in the [Administration and Configuration Guide](https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/) for Red Hat JBoss Enterprise Application Platform._


### REST Endpoints on OpenShift

* Create task

  ```
  curl -i -u 'redhat:redhat1!' -H "Content-Length: 0" -X POST http://tasks-dev.10.1.2.10.xip.io/ws/tasks/task1
  ```

* Get a task by id

  ```
  curl -u 'redhat:redhat1!' -H "Accept: application/json" -X GET http://tasks-dev.10.1.2.10.xip.io/ws/tasks/1
  ```

* Get all user tasks

  ```

  curl -u 'redhat:redhat1!' -H "Accept: application/json" -X GET http://tasks-dev.10.1.2.10.xip.io/ws/tasks
  ```

* Delete a task by id

  ```
  curl -i -u 'redhat:redhat1!' -X DELETE http://tasks-dev.10.1.2.10.xip.io/ws/tasks/1
  ```

* Generate CPU load. Last parameter is duration of load in seconds

  ```
  curl -X GET http://tasks-dev.10.1.2.10.xip.io/demo/load/5 # 5 seconds
  ```
