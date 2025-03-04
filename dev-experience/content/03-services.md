### Services
![Image showing an application with an arrow representing a ServiceBinding to a Postgres Database](./images/services.png)

It's fairly typical that an application relies on some externally managed resources like databases, OAuth servers, caches, messaging servers and others to run.  Tanzu Platform for Kubernetes provides a way for platform teams to consolidate all the services that have the best support in your organization into a single catalog and enables application developers to consume those services by simply binding them to their applications.  

Let's explore binding platform-managed and externally-managed services to our application.

All of the service management commands are grouped under the `tanzu service` category in the CLI.  Call the following command to see the types of operations you can perform.
```
tanzu service --help
```

![Image showing a Space containing a list of service types](./images/service-type-list.png)

We can use the Tanzu CLI to have a look at the catalog of service types our platform team and service providers have made available for us.  This list is a curated list, and could be sourced from in cluster deployments, cloud-provider SaaS services, or really anything.  The nice thing is that you don't have to care how those services are managed.  You can consume them all in the same way using the flow we'll go through next.  

First, let's call the following CLI command to view the catalog of service types our platform has for us to use.
```
tanzu service type list
```

Our application can use PostgreSQL databases, and our platform team has made that type of service available to us.  Let's create an instance of that service so that we can use it with our app.  The `tanzu service create TYPE/NAME` command allows us to create a service of a specified TYPE and give it the name of NAME.  As you can see in the list of service types, we have a type called `PostgreSQLInstance` that looks promising.  Let's create an instance of that called `my-db` with the following command.
```
tanzu service create PostgreSQLInstance/my-db --skip-bind-prompt
```

You can also use interactive prompt to provision a new service or attach a pre-provisioned service
![Provision service](./images/service.png)

Great! Now we can have a look at the list of services in our space. We should only have the one we just created called `my-db`.
```
tanzu service list
```

We can get some details about our service by using the `tanzu service get` command.
```
tanzu service get PostgreSQLInstance/my-db
```

In the output, we can see a section called `PostgreSQLInstance type spec`.  These values are configurable items that our platform team allows us to specify when we create the service. These values have some defaults that are applied if we don't specify anything, but we can selectively choose different configurations if we need to.  

For example, maybe we want to have a database that allows more than 1 GB of storage. We could have called the create command like this (don't execute this) `tanzu service create PostgreSQLInstance/my-db --parameter storageGB=10` to have created a PostgreSQL instance with 10GB of storage. Each service type will have different configuration options exposed, so you might need to talk with your platform team to find out all the options.

![Image showing a Service binding connected to an application and Postgres service and a set of secret values getting injected into the application](./images/service-binding.png)

At this point, we have a PostgreSQL database, but our app doesn't know anything about it. We can change that by *binding* our database instance to our application. Binding associates the instance of a service with an application, and injects information about that service using [Service Bindings for Kubernetes](https://servicebinding.io/).  The specification standardizes how service information is injected into workloads via a specific directory structure and files mounted into a container application.

[Many libraries](https://servicebinding.io/application-developer/) know how to interpret this standardized information, and some like [Steeltoe](https://docs.steeltoe.io/api/v3/connectors/) and [Spring Cloud Bindings](https://github.com/spring-cloud/spring-cloud-bindings) can automatically inject configuration to make consuming those services very easy.

We're working with a Spring Boot application, and our platform team is using the [Spring Boot Cloud Native Buildpack from Packeto](https://github.com/paketo-buildpacks/spring-boot) to automatically add the Spring Cloud Bindings library to our application library path.  This means that our database configuration will be automatically injected and beans will be automatically created to use the bound database when our application starts up. Let's bind the service to our application.
```
tanzu service bind PostgreSQLInstance/my-db ContainerApp/inclusion --as db
```

You can also follow interactive prompt
![Bind service](./images/binding.png)

Let's refresh our application and see that the emoji and the text in the header have changed for our app (from "powered by H2" to "powered by POSTGRESQL"). ![Image showing change to Inclusion app to show "powered by PostgreSQL" instead of "powered by H2"](./images/inclusion-postgres-binding.png)

If you don't see a change immediately, retry after waiting for 1 minute or so.  Changes to the application are rolled out gracefully, and load is not shifted to the new version of your application until it is healthy.  You can go to https://www.mgmt.cloud.vmware.com/hub/application-engine/space/details/{{< param  session_name >}}/topology to see the URL for your application if you accidentally closed the tab for it.  Click on the "Space URL" link at the upper middle of the page.

![Image showing a Service binding connected to an application and PreProvisonedService object and a set of secret values getting injected into the application.  The application is shown connected to an externally managed Postgres database](./images/preprovisioned-service.png)

The PostgreSQL service we are using is provisioned using automation for us in the platform. But what if we have a database that is managed by another team, or an existing database that we need to keep using? Don't worry! We can still use the service binding mechanism to simplify this process. We're going to switch to a shared PostgreSQL database that all the workshop attendees can use together.  If we are careful about how we format the information for the binding, we can even still use the Spring Cloud Bindings library to automatically configure the application for us.

If we have a look at the [PostgreSQL section in the Spring Cloud Bindings README.md](https://github.com/spring-cloud/spring-cloud-bindings?tab=readme-ov-file#postgresql-rdbms), we can see that we need to include a `username`, and `password` setting.  We then can either specify a `jdbc-url` setting or the `host`, `port`, `database` values.  We can optionally add in additional configuration with the `sslmode`, `sslrootcert`, and `options` values if we need to fine-tune the connection. 


Great! The platform will restart our application to get it to pick up on the new binding. Let's refresh our application and see that the emoji has changed again for our app.  And as other users use the shared database, you'll start to see more emojis show up.

Remember in the section where we dove deeper into the application configuration and added contact metadata to our app?  We can also add information for application operators who need to deploy our app to other environments about the service bindings our application supports.  We can use `tanzu app config servicebinding` command to add information about the name of the binding and the types of services we support. First, let's display the service catalog on our platform again.

```
tanzu service type list
```

Notice that the output shows a `TYPE` column and a `BINDING TYPE` column.  We want to make sure to specify the value of the `BINDING TYPE` for the `TYPE` of service we allow.  So, since we support the `PostgreSQLInstance` service type, we'll want to specify `postgresql` in our `tanzu app config servicebinding` command.  Let's add a service binding reference with the alias `db` and binding type of `postgresql` to our application configuration.
```
tanzu app config servicebinding set db=postgresql
```

Now, have a look at the `inclusion/.tanzu/config/inclusion.yml` file to see the new binding reference.
```
cat ~/inclusion/.tanzu/config/inclusion.yml
```
We can see the reference to our accepted service binding!  If our application supports multiple types of databases, we could add more types to the yaml file directly, or we can call the `tanzu app config servicebinding set db=<new-type>` command again replacing the `<new-type>` text with the additional binding type we support.

Fantastic!  We were able to create a PostgreSQL service instance for our application that has all the policies and best practices defined for our environment and delete the database when we didn't need it anymore.  We were also able to inject information about that service into our application using service bindings and remove the binding when we no longer needed it. We were able to add an externally managed service via a Secret and bind that information into our application.  We were also able to add information about the bindings and types our application supports for application operators that deploy our application to other environments.  And we were able to do all that with no code changes to our application!

Let's move on to explore scaling our application with Tanzu Platform for Kubernetes.