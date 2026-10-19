## @Entity 
@Entity is simply tells that the below class fields maps to table of some db and its objects will be mapped to rows of the table.  
@Entity also requires a no args constructor similar to jackson to create objects and the data are injected through reflections or field.

### Request Flow to controller 
Request with JSON data, it is converted into java object. and @Valid makes validation checks and after that it proceeds to controller

### How Jackson Converts JSON to java object
It creates object with No Arguments Constructor and injects with setters or fields.  
So NoArgsConstructor is necessary and setters are good practise. (Jackson can access privates through reflections(need not to think for now))


## Methods provided by Spring JPA (jakarta persistence api)
1. create/read -> save()
2. search many or one-> findAll() and find()
3. delete -> deleteById()
4. exists -> existsById()

### Why a interface in repository layer ? 
if there would be a class we have to manually write the query logic. Interface we define. common signatures are already provided like (listed above).
and custom method signatures we define inside interface.
at runtime JPA provides us the implementation of the repository layer methods implementations.

### Error when running newly created spring app due to @AutoConfig of database connection ?
@SpringBootApplication(exclude = {DataSourceAutoConfigiration.class})

### Configuration of application.properties for DB connection 
```text
spring.datasource.url=`url`/dbname
spring.datasource.username=***
spring.datasource.password=***
spring.jpa.hibernate.ddl-auto=***

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

