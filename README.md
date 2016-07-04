#Catatumbo - Object Mapping and Persistence framework for Google Cloud Datastore 
Catatumbo is an Open Source persistence framework for mapping Java objects (POJOs) to 
[Google Cloud Datastore](https://cloud.google.com/datastore/) and vice versa. 

Catatumbo framework is built on top of 
[gcloud-java-datasource API](https://github.com/GoogleCloudPlatform/gcloud-java/tree/master/gcloud-java-datastore). 
The framework provides a handful of annotations to make your model classes manageable by the framework. In other words, 
Catatumbo is similar to JPA (Java Persistence API), but is specifically designed to work with Google Cloud Datastore 
instead of Relational Databases.  

> **Note: Catatumbo is a work-in-progress and future changes are not guaranteed to be backward compatible**

##Features
* Automatic mapping of model classes (POJOs) to Cloud Datastore Entities and vice versa
* Automatic generation of Identifiers (for both Numeric and String types)  
* Support for a variety of Data Types - boolean, Boolean, char, Character, short, Short, int, Integer, long, Long, float, Float, double, Double, String, Byte Arrays, Char 
Arrays, Date, Calendar, Keys/Key References
* Create, Update, Delete a single object or multiple objects with a single method call
* Execute GQL Queries. 
* Transaction Support  

##Annotations 
###@Entity (`com.jmethods.catatumbo.Entity`)
The `@Entity` annotation is used to annotate model classes that need to be persisted. Can optionally define the Kind, 
if it needs to be different than the class name.

The below example marks the Person class as a managed Entity and maps the Person objects to entities of Kind Person in 
the Google Cloud Datastore.  

```java

	import com.jmethods.catatumbo.Entity;
	
	@Entity
	public class Person {
		//members of Person go here. For example, id, firstName, lastName and their corresponding accessor methods. 
	}

```

If you need to map Person objects to entities of Kind People in the Google Cloud Datastore, simply specify the kind as shown below: 

```java

	import com.jmethods.catatumbo.Entity;
	
	@Entity(kind="People")
	public class Person {
		//members of Person go here. For example, id, firstName, lastName and their corresponding accessor methods. 
	}

```

###@Identifier (`com.jmethods.catatumbo.Identifier`)
The `@Identifier` annotation is used to mark the identifier of your model class. In a given class, only one field can 
have this annotation. The field with this annotation must be of type long, Long or String. Identifiers can either be 
auto generated by the framework or managed by the application. 

The example below marks the field `id` as an identifier and tells that the identifiers have to be automatically 
generated by the framework, when needed.  

```java

	import com.jmethods.catatumbo.Entity;
	import com.jmethods.catatumbo.Identifier;
	
	@Entity
	public class Person {
		@Identifier
		private long id;
		//Other members of Person go here. For example, firstName, lastName and their corresponding accessor methods. 
	}

```

If you rather want your application to manage the identifiers (e.g. use email address as the identifier), you can 
simply set the autoGenerated to false as shown below: 

```java

	import com.jmethods.catatumbo.Entity;
	import com.jmethods.catatumbo.Identifier;
	
	@Entity
	public class Person {
		@Identifier(autoGenerated=false)
		private String emailAddress;
		//Other members of Person go here. For example, firstName, lastName and their corresponding accessor methods. 
	}

```

###@Property (`com.jmethods.catatumbo.Property`)
By default, all instance variables that have corresponding accessor methods would be persisted by Catatumbo. The 
property name in the Datastore would be same as the name of the instance variable. If you need to map an instance 
variable to a different property name in the Cloud Datastore, you would use this annotation. In addition, this 
annotation controls whether or not a field is indexed. 

The example below maps the firstName field of the Person entity to a property names fname. 

```java

	import com.jmethods.catatumbo.Entity;
	import com.jmethods.catatumbo.Identifier;
	
	@Entity
	public class Person {
		@Identifier(autoGenerated=false)
		private String emailAddress;
		
		@Property(name="fname")
		private String firstName;
		//Other members of Person go here. For example, lastName, date of birth and their corresponding accessor methods. 
	}

```

###@Ingore (`com.jmethods.catatumbo.Ignore`)
Use this annotation on any field that should be excluded from persistence. 

```java

	import com.jmethods.catatumbo.Entity;
	import com.jmethods.catatumbo.Identifier;
	
	@Entity
	public class Person {
		@Identifier(autoGenerated=false)
		private String emailAddress;
		
		@Property(name="fname")
		private String firstName;
		//Other members of Person go here. For example, firstName, lastName and their corresponding accessor methods. 
		
		private Date dateOfBirth;
		
		@Ignore 
		private int age; //Computed based on the date of birth, so no need to store it in the Datastore. 
	}

```

##Example Code for CRUD Operations 

###Setting up your project 
The easiest way to include Catatumbo into your project is by adding the below dependency to your Maven project: 

```xml

	<dependency>
	  <groupId>com.jmethods</groupId>
	  <artifactId>catatumbo</artifactId>
	  <version>0.1.0</version>
	</dependency>

```

This will download the Catatumbo JAR files as well as all necessary dependencies into your project. 

###Create your Model Class 

We will use the below Person class to demonstrate how to use Catatumbo. 

```java

	import com.jmethods.catatumbo.Entity;
	import com.jmethods.catatumbo.Identifier;
	import com.jmethods.catatumbo.Property;
	
	@Entity(kind = "people")
	public class Person {
	
		@Identifier
		private long id;
	
		@Property(name = "fname")
		private String firstName;
	
		@Property(name = "lname")
		private String lastName;
	
		@Property(indexed = false)
		private int birthYear;
	
		private boolean citizen;
	
		public long getId() {
			return id;
		}
	
		public void setId(long id) {
			this.id = id;
		}
	
		public String getFirstName() {
			return firstName;
		}
	
		public void setFirstName(String firstName) {
			this.firstName = firstName;
		}
	
		public String getLastName() {
			return lastName;
		}
	
		public void setLastName(String lastName) {
			this.lastName = lastName;
		}
	
		public int getBirthYear() {
			return birthYear;
		}
	
		public void setBirthYear(int birthYear) {
			this.birthYear = birthYear;
		}
	
		public boolean isCitizen() {
			return citizen;
		}
	
		public void setCitizen(boolean citizen) {
			this.citizen = citizen;
		}
	
	}
	
```
	
###Explanation
* By annotating the class with `@Entity` annotation, you are indicating that 
the EntityManager is allowed to manage the persistence of Person objects. 
* Each Entity class must have a field with an annotation of `@Identifier`. In 
our example, we have indicated that field `id` is the Identifier. 
* Field `firstName` is annotated with `@Property`. We are also indicating that 
`firstName` be mapped to a property named `fname` in the Cloud Datastore. 
* We are explicitly excluding `birthYear` from being indexed. 
* Lastly, field `citizen` does not have any annotations, but it will still be 
persisted by the EntityManager. The property name in the Datastore will also 
be `citizen`. 

Now that you have your model class, saving or loading Person objects is a snap.

###Creating the EntityManager 
The first and foremost thing is to create an EntityManager to manage the 
persistence. The code below shows how to create an EntityManager.

```java 

	EntityManagerFactory emf = EntityManagerFactory.getInstance();
	EntityManager em = emf.createEntityManager("your-google-cloud-project-id",
			"/path/to/your/credentials.json");

```

Once you have the EntityManager, you can insert, update, delete, load, run 
queries etc. 

###Inserting a Person object

```java

	//Create a Person object and set the desired values. 
	Person person = new Person();
	person.setFirstName("John");
	person.setLastName("Doe");
	person.setBirthYear(1975);
	person.setCitizen(true);

	//Insert the person. ID is automatically generated. You can also set your 
	//own ID if auto generation is not needed. 
	person = em.insert(person);
	System.out.printf("person with ID %d created successfully", person.getId());

```
	
When you run this code, you should see something similar to the below line in your console: 

	person with ID 5668906396024832 created successfully 

###Loading a Person object

```java

	// Load a Person object
	Person person = em.load(Person.class, 5629499534213120L);
	System.out.printf("First Name: %s; Last Name: %s; Birth Year: %d; Citizen: %s\n", person.getFirstName(),
			person.getLastName(), person.getBirthYear(), person.isCitizen());

```
	
In the above example, we are asking the EntityManager to load an entity with ID 5668906396024832. We are also telling the EntityManager that we are expecting the result to be a Person object. When you run this sample, you should see the below line in console: 

	First Name: John; Last Name: Doe; Birth Year: 1975; Citizen: true
	
###Updating a Person Object 

```java

	//First load the Person object that needs to be updated 	
	Person person = em.load(Person.class, 5629499534213120L);
	//Make changes to the Person object 
	person.setLastName("Smith");
	person.setCitizen(false);
	//Update the datastore. 
	em.update(person);
	//Reload the person object
	Person updatedPerson = em.load(Person.class, 5629499534213120L);
	System.out.printf("First Name: %s; Last Name: %s; Birth Year: %d; Citizen: %s\n",
			updatedPerson.getFirstName(), updatedPerson.getLastName(), updatedPerson.getBirthYear(),
			updatedPerson.isCitizen());

```

###Deleting a Person with a given ID

```java
 
	em.delete(Person.class, 5668906396024832L);
	
```

###Deleting a Person Object 

```java

	Person person = em.load(Person.class, 5629499534213120L);
	em.delete(person);

```

###Querying 
Querying is currently implemented with GQL. 

####Simple Select All Query

```java

	EntityQueryRequest request = em.createEntityQueryRequest("SELECT * FROM people");
	QueryResponse<Person> response = em.execute(Person.class, request);
	List<Person> persons = response.getResults();
	//Process the list as needed
	
```

####Query with a Filter using Positional Parameters

```java

	// Select with positional bindings (parameters). Note the @1 in the query. 
	EntityQueryRequest request = em.createEntityQueryRequest("SELECT * FROM people WHERE lname=@1");
	//Add a binding - in this case @1 will be set to "Smith"
	request.addPositionalBinding("Smith");
	QueryResponse<Person> response = em.execute(Person.class, request);
	List<Person> persons = response.getResults();
	// Process the list as needed

```

####Query with a Filter using Named Parameters

```java

	// Select with named bindings (parameters). Note the @citizen in the query, which is a named parameter. 
	EntityQueryRequest request = em.createEntityQueryRequest("SELECT * FROM people WHERE citizen = @citizen");
	//Set the parameter citizen to the desired value and execute the query
	request.addNamedBinding("citizen", false);
	QueryResponse<Person> response = em.execute(Person.class, request);
	List<Person> persons = response.getResults();
	// Process the list as needed

```

####Pagination Example 

```java

	final String baseQuery = "SELECT * FROM people ORDER BY __key__ LIMIT @Limit";
	String query = baseQuery;
	EntityQueryRequest request = em.createEntityQueryRequest(query);
	request.setNamedBinding("Limit", 5);
	QueryResponse<Person> response = em.execute(Person.class, request);
	List<Person> persons = response.getResults();
	// Process the first page of results as needed...
	
	// Get Next Page.  
	query = baseQuery + " OFFSET @Offset";
	request.setQuery(query);
	//Set the offset to the endCursor from previous response. 
	request.setNamedBinding("Offset", response.getEndCursor());
	response = em.execute(Person.class, request);
	persons = response.getResults();
	//Process the second page as needed. 

```







