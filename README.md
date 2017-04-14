## A Thousand and One Way to deploy OpenShift Application

The hardest thing when starting OpenShift from a developer point of view is not in deploying an application to it. This step can be done very easily. We found indeed that the most challenging part is to find the best way to do it ! Because OpenShift is so adaptable and flexible that it offers _A Thousand and One Way_ to deploy an application. The _best way_ will be the one that best fit a bunch of criteria:
* Your development process: what is your existing ecosystem and flow? (already using Jenkins, Nexus, ...)
* Your developer's maturity regarding Containers and CI/CD: do they want to manage Containers Dockerfile ou Pipeline as code ?
* Your Ops or Technical exports involvement: wether they need to customize provided base images or base templates for introducing specificities,
* Your will to build different Service offers allowing multiple workflows adapted to multiple development teams.

Following this tutorial, you will deploy the same application using 6 different methods. Remember that this is _just_ 6 methods that represents main ways of doing thins, there's a number of variations that may be defined using some elements picked from one or another method exposed here.

For deploying the application repeatedly, you will only need an OpenShift project for isolating things out. Methods #6 may also require a `JBoss Developer Studio` on version 10+. We propose you create this project using the identifier `ocp-tasks` because some methods have this id hard-coded.

So startup by creating the project using the following command:
```
oc new-project ocp-tasks --display-name="OpenShift Tasks" --description="Demonstrating many ways to deploy application on OpenShift"
```
and here we go!

### #1: Use provided JBoss EAP 7 S2I template

This is the most obvious for people starting with OpenShift: you'll use the default template and base image provided by OpenShift without any customization upfront. To illustrate this flow within your `ocp-tasks` project:
* Go to "Add to project" page,
* Pick the `jboss-eap70-openshift:1.4` template,
* Adapt the name of application to `tpl-tasks` or something,
* Refer this Github repository (`http://github.com/lbroudoux/openshift-tasks`) or another internal one,
* Optionally, add some extra environment variables on the BuildConfig for referring Maven component mirror (`MAVEN_MIRROR_URL`),
* Hit the save button and you'll got the application running in few minutes.

![base-template-s2i](https://raw.githubusercontent.com/lbroudoux/openshift-tasks/master/assets/base-template-s2i.png)  

### #2: Bring your own S2I template

This variant consists in providing your own template that will be a customization of OpenShift provided template. Providing your own template allows you to lower the number of parameters the developer will have to fill and fixing hard values. To illustrate this flow within your `ocp-tasks` project:
* Register your app template in OpenShift using `oc create -f https://raw.githubusercontent.com/lbroudoux/openshift-tasks/master/app-template.yaml -n ocp-tasks`,
* Go to "Add to project" page,
* Pick the `openshift-tasks` template,
* Adapt the name of application to `s2i-tasks` or something,
* Refer this Github repository (`http://github.com/lbroudoux/openshift-tasks`) or another internal one,
* Hit the save button and you'll got the application running in few minutes.

![base-template-s2i](https://raw.githubusercontent.com/lbroudoux/openshift-tasks/master/assets/custom-template-s2i.png)  

### #3: Custom S2I for binary deployment

This variant consists in providing your own template like the previous one but adapting the Source-to-image process by making OpenShift build your application image not from source but from a binary file you will inject.

To illustrate this flow within your `ocp-tasks` project, we assume that previous step has been done and template is already registered:
* Go to "Add to project" page,
* Pick the `openshift-tasks` template,
* Adapt the name of application to `bin-tasks` or something,
* Refer the companion Github repository (`http://github.com/lbroudoux/openshift-tasks-bin-deploy`) or another internal one,
* Hit the save button.

![s2i-binary-deployment](https://raw.githubusercontent.com/lbroudoux/openshift-tasks/master/assets/s2i-binary-deployment.png)  

### #4: Build pipeline managed out of Source code


### #5: Build pipeline managed within Source code

![jenkinsfile-template](https://raw.githubusercontent.com/lbroudoux/openshift-tasks/master/assets/jenkinsfile-template.png)

### #6: Directly from IDE



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
