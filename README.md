Step 1 \
Review the base structure of the application
For your convenience, this scenario has been created with a base project using the Java programming language and the Apache Maven build tool. Initially the project contains a couple web-oriented files which are out of scope for this module. See the Spring MVC / Rest Services module for more details. These files are in place to give a graphical view of the Database content.

As you review the content you will notice that there are a couple TODO comments. Do not remove them! These comments are used as markers for later exercises in this scenario.

In the next section we will add the dependencies and write the class files necessary to run the application.

Step 2 \
Read content from a database
In Step 1 you learned how to get started with our project. In this step, we will add functionality for our Fruit basket application to display content from the database.

1. Adding JPA (Hibernate) to the application

Since our application will need to access a database to retrieve and store fruit entries we need to add a Database Connection library. One such implementation in Spring is the Spring Data project which contains the Java Persistence APIs (JPA) and a JPA implementation - Hibernate. Hibernate has been tested and verified as part of the OpenShift Application Runtimes so we are going to use it here. Spring Boot has full knowledge of these libraries as well so we get to take advantage of Spring Boot's auto-configuration with these libraries as well!

To add Spring Data + JPA and Hibernate to our project all we have to do is to add the following line in pom.xml

Copy to Editor    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
      </dependency>
  
We also need a Database to actually interact with. When running locally or when running tests an in-memory Database is often used over connection to an external Database because its lifecycle can be managed by Spring and we don't have to worry about outages impacting our local development. H2 is a small in-memory database that is perfect for testing but is not recommended for production environments. To add H2 add the following dependency at the comment <!-- TODO: Add H2 database dependency here --> in the local profile.

Copy to Editor    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
      </dependency>
  
If Spring Boot sees a database like H2 on the classpath it will automatically configure an in-memory one for us as well as all the connection Beans necessary to connect to it. We've chosen to override these settings in the src/main/resources/application-local.properties file to demonstrate that you can interact with Spring Boot's auto-configuration quite easily.

2. Create an Entity class

We are going to implement an Entity class that represents a Fruit. This class is used to map our object to a database schema.

First, we need to create the java class file. For that, you need to click on the following link which opens the skeletal class file in the editor: src/main/java/com/example/service/Fruit.java

Then, copy the below content into the file (or use the Copy to Editor button):

Copy to Editorpackage com.example.service;
  
  import javax.persistence.Entity;
  import javax.persistence.GeneratedValue;
  import javax.persistence.GenerationType;
  import javax.persistence.Id;
  
  @Entity
  public class Fruit {
  
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;
  
      private String name;
  
      public Fruit() {
      }
  
      public Fruit(String type) {
          this.name = type;
      }
  
      public Long getId() {
          return id;
      }
  
      public void setId(Long id) {
          this.id = id;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  }
  
The @Entity annotation marks the object as a persistable Entity for Spring Data. The @Id and @GeneratedValue annotations are JPA annotations which mark the the id field as the database ID field which has an auto-generated value. Spring provides the code which makes these annotations work behind the scenes.

3.Create a repository class for our content

The repository should provide methods for inserting, updating, reading, and deleting Fruits from the database. We are going to use Spring Data for this which already provides us with a lot of the boilerplate code, so all we have to do is to add an interface that extends the CrudRepository<T, I> interface provided by Spring Data.

First, we need to fill out the skeltal java class file. For that, you need to click on the following link which opens the file in the editor: src/main/java/com/example/service/FruitRepository.java

Then, copy the below content into the file (or use the Copy to Editor button):

Copy to Editorpackage com.example.service;
  
  import org.springframework.data.jpa.repository.Query;
  import org.springframework.data.repository.CrudRepository;
  import org.springframework.stereotype.Repository;
  
  import java.util.List;
  
  @Repository
  public interface FruitRepository extends CrudRepository<Fruit, Long> {
  // TODO query methods
  }
  
4. Populate the database with initial content

To pre-populate the database with content, Hibernate offers a nifty feature where we can provide an SQL file that populates the content.

First, we need to create the SQL file. For that, you need to click on the following link which opens the empty file in the editor: src/main/resources/import.sql

Then, copy the below content into the file (or use the Copy to Editor button):

Copy to Editorinsert into fruit (name) values ('Cherry');
  insert into fruit (name) values ('Apple');
  insert into fruit (name) values ('Banana');
  
5. Test and Verify To verify that we can use the FruitRepository for retrieving and storing Fruit objects we have created a JUnit Test Class at src/test/java/com/example/service/DatabaseTest.java

Take a bit of time and review the tests. The testGetAll test will return all fruits in the repository, which should be three because of the content in import.sql. The getOne test will retrieve the fruit with ID 1 (e.g., the Cherry) and then check that it's not null. The getWrongId checks that if we try to retrieve a fruit id that doesn't exist and check that fruitRepository returns null.

We can now test that our FruitRepository can connect to the data source, retrieve data and Run the application by executing the below command:

mvn verify

In the console you should now see the following (if the output is too noisy the first time around because of downloads simply run the command again):

Results :
  
  Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
  
NOTE: As a reminder: the configuration for database connectivity is found in the application properties files in src/main/resources/ since we chose to override the Spring Boot defaults. For local we use the application-local.properties file. On OpenShift we use the application-openshift.properties file.

6. Review The Controller

To see how this repository could be used in a web application we're going to quickly review the FruitController file. Open src/main/java/com/example/service/FruitController.java in your editor.

In order to use our FruitRepository we must first autowire one in our constructor:

@Autowired
  public FruitController(FruitRepository repository) {
      this.repository = repository;
  }
  
Since Repositories are a managed Bean in Spring we have to tell Spring to inject an instance into our Controller for use. We use Constructor Autowiring as per the suggestion of the Spring Team since it is considered best practice and allows easier mock injecting in tests.

If we are fetching all records from the database we use the aforementioned repository.findAll() method. To delete we have repository.delete(id), to save a new entry we have repository.save(fruit), and so on. All these methods reside on the CrudRepository interface which is automatically implemented by Spring.

NOTE: The usual input validations have been omitted from this class for brevity's sake. Always validate and sanitize input coming from the client!

7. Query Methods

The methods provided by CrudRepository are nice when we are dealing directly with IDs but sometimes we don't have that information. Let's say, for example, we have a search box that allows users to search by Fruit Name. We currently do not support that functionality without using a findAll() and then filtering the results in the application. Not a good idea for large datasets.

Fortunately there exists within the JPA specification a section on Query Methods. Query methods are methods on Repositories that follow a specific naming pattern. These methods can then be turned into implementation code by JPA providers for running actual queries.

Let's re-open the FruitRepository class file in the editor: src/main/java/com/example/service/FruitRepository.java and add the following code at the TODO line (or use the Copy to Editor button):

Copy to Editor    List<Fruit> findByName(String name);
  
      default List<Fruit> findAllFruitsByName(String name) {
          return findByName(name);
      }
  
      @Query("select f from Fruit f where f.name like %?1")
      List<Fruit> findByNameLike(String name);
  
The first method findByName(String name) is a standard JPA Query Method. The format of this method is findBy followed by the field we are querying on for our Fruit object. For the name field we use the same name. Note that the field name is capitalized to follow the standard Java camelCase format.

The format of these methods can get pretty complex but here we're only referring to the name field of our Fruit model so the method name is pretty short. For a more in depth guide to Query Methods see this article.

The second method demonstrates default methods which were introduced in Java 8. This allows us to provide a method definition for an interface method. In this case we've created an alias for the findByName() method which simply delegates to the JPA Query Method. This technique is particularly useful when the Query Methods get very complex and long as it allows us to define more readable repository methods that delegate to their more complex counterpart. We can also aggregate operations in this way.

The last method is an example of the @Query annotation which allows you to provide an actual SQL Query to execute. Note, however, that the syntax is a little different. This is actually a dialect called JPQL which looks like ANSI SQL but with a few differences. You can, however, use native queries by adding the nativeQuery=true argument to the annotation. Be aware that this approach can cause tight coupling between your code and your database if you use database-specific extensions that won't be caught until runtime. This is, however, particularly useful for complex queries that are better suited for plain native SQL.

8. Verify the application

To verify that the application actually works now we need to actually run the application. Run the application by executing the following command mvn spring-boot:run

Next, click on the Local Web Browser tab in the console frame of this browser window which will open another tab or window of your browser pointing to port 8080 on your client. Or use this link.

You should now see an HTML page with an Add a Fruit textbox on the left and a Fruits List view on the right with the three fruits we pre-populated with the import.sql file. Adding a new fruit name into the textbox and clicking Save should insert the new Fruit name into the right-hand list. Clicking on the Remove buttons should also work as expected.

4. Stop the application

Before moving on, click in the terminal window and then press + to stop the running application!

Congratulations
You have now learned how to create and test a data repository that can create, read, update and delete content from a database. We have so far been testing this with an in-memory database, but later we will replace this with a full blow SQL server running on OpenShift, but first, we should create REST services that the web page can use to update content.

Step 3
Deploy to OpenShift Application Platform
For running locally the H2 Database has been a good choice, but when we now move into a container platform we want to use a more production-like database, and for that, we are going to use PostgreSQL.

Before we deploy the application to OpenShift and verify that it runs correctly, there are a couple of things we have do. We need to add a driver for the PostgreSQL database that we are going to use, and we also need to add health checks so that OpenShift correctly can detect if our application is working.

1. Login to OpenShift Container Platform

To login, we will use the oc command and then specify username and password like this:

oc login 2886795301-8443-kitek02.environments.katacoda.com --insecure-skip-tls-verify=true -u developer -p developer

Now let's create a new project

oc new-project dev --display-name="Dev - Spring Boot App"

2. Create the database

Since this is your own personal project you need to create a database instance that your application can connect to. In a shared environment this would typically be provided for you, that's why we are not deploying this as part of your application. It's however very simple to do that in OpenShift. All you need to do is to execute the below command in the console.

oc new-app -e POSTGRESQL_USER=dev \
               -e POSTGRESQL_PASSWORD=secret \
               -e POSTGRESQL_DATABASE=my_data \
               openshift/postgresql-92-centos7 \
               --name=my-database

This command creates a new deployable Postgres instance using the OpenShift Postgresql image named my-database. We can check the status of the deployment by running oc status as mentioned in the output of the above command.

When deployed you should see similar output to the following:

$ oc status
  In project Dev - Spring Boot App (dev) on server https://172.17.0.85:8443
  
  svc/my-database - 172.30.167.58:5432
    dc/my-database deploys istag/my-database:latest
      deployment #1 deployed about a minute ago - 1 pod
  
We can see here that 1 pod is deployed with our Database image and it is now ready to consume.

3. Review Database configuration

Take some time and review the src/main/fabric8/deployment.yml.

As you can see that file specifies a couple of elements that are needed for our deployment. It also uses the username and password from a Kubernetes Secret. For this environment we are providing the secret in this file src/main/fabric8/credentials-secret.yml, however in a production environment this would likely be provided to you by the Ops team.

Now, review the src/main/resources/application-openshift.properties

In this file we are using the configuration from the deployment.yml to read the username, password, and other connection details.

4. Add the PostgreSQL database driver

So far our application has only used the H2 embedded Database. We now need to add a dependency for the PostgreSQL driver. We do that by adding a runtime dependency under the openshift profile in the pom.xml file.

Copy to Editor        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>${postgresql.version}</version>
            <scope>runtime</scope>
          </dependency>
  
5. Add a health check

We also need a health check so that OpenShift can detect when our application is responding correctly. Spring Boot provides a nice feature for this called Actuator, which exposes health data under the path /health. All we need to do is to add the following dependency to pom.xml at the TODO comment..

Copy to Editor    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
  
6. Deploy the application to OpenShift

Run the following command to deploy the application to OpenShift

mvn package fabric8:deploy -Popenshift -DskipTests

This step may take some time to do the Maven build and the OpenShift deployment. After the build completes you can verify that everything is started by running the following command:

oc rollout status dc/rhoar-training

Then either go to the OpenShift web console and click on the route or click here

Make sure that you can add and remove fruits using the web application.

Congratulations
You have now learned how to deploy a Spring Boot application to OpenShift Container Platform with a PostgreSQL database. Click Summary for more details and suggested next steps.