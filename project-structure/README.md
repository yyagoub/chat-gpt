# Components:

This project combined two main components: the front-end and the back-end.

The front-end is built using ReactJS as a UI framework, React Router to route between pages, Axios for API calls, and Redux for state management.

The back-end is built using Spring Boot as a framework, Spring Data for data management, Spring Web for exposing services as APIs, Liquibase for data migrations, and Spring Security to secure the APIs.




# Project Structure:
The style used to build this project is Domain-Driven Design. The project is split into packages, and each package represents a domain, usually, a table behind this domain.

This structure is used in both the front-end and back-end components. The back-end focuses on business and data management, while the front-end focuses on representing and manipulating the data.
## The back-end domain structure:

each domain will have these classes as a root of behavior-based, such as:

#### 1. Entity: 

1. represents the table, which is a heavy form of the table's data, 
2. that should be used only for querying and fetching data; 
3. it should not be passed between services.
4. Naming of an entity should have the table name without the prefix for example:
table name is TBL_IAM_CUSTOMER_PROFILE then entity name is CustomerProfile
5.  only soft delete by set deleted column value to 1
6. every entity must have a where condetiom to remove deleted: @Where(clause = "deleted = 0")
7. Lombok @Data used with every entity 
8. @ToString.Exclude used with every complex object or entity relation
9. extends EntityMeta in every entity to enhirt AuditingMetadata columns
```
Java code example
```

#### 2. DTO: 
1. represents the light form of the entity's data, which can be passed between services.
2. Naming of a DTO should have the Entity name with a suffex for example:
Entity name is CustomerProfile then DTO name is CustomerProfileDto
3. Lombok @Data used with every DTO 
4. @ToString.Exclude used with every complex object or DTO relation
5. extends DtoMeta in every entity to enhirt AuditingMetadata columns

#### 3. Model: 
1. represents the client request; 
2. the main purpose of this is to control the data passed by the requester.
2. Naming of a Model should have the Entity name with a suffex for example:
Entity name is CustomerProfile then Model name is CustomerProfileModel
3. Lombok @Data used with every Model 
4. @ToString.Exclude used with every complex object or Model relation
5. extends ModelMeta in every entity to enhirt AuditingMetadata columns

#### 4. Mapper: 
1. an interface that is stateful to all mapping happening within a domain
2. it may map the domain's entity or its joined entity.
3. Mapper is annotated with @Mapper 
4. extends interface StructMapper <E extends EntityMeta, M extends ModelMeta, D extends DtoMeta>
5. StructMap framework will implement the interfaces into classes.
6. Mapper must not map chiled/relation objects or lists.
7. if there is a complex objects it should be handled separatly to prevent stackoverslow.
8. there is no bean injection in a mapper class as there must not have a dependency ever.

#### 5. Repository: 
1. All data access and manipulation should be written in this class.
2. write your query for readability.
3. extends ExtendedRepository<T extends EntityMeta, ID extends Serializable>
4. you should not use hard delete.
5. use soft delete method for delete: ExtendedRepository.delete(final Long id)

#### 6. Validator: 
1. All validation should be in this class; 
2. no other classes should throw an exception except for the Validator.
#### 7. UnprocessableEntityException: 
1. represents any business validation error.
#### 8. Service: 
1. service is presenting a story of how this service interacts with different behavior classes, such as validate request, then mapping to a DTO, then saving as an entity, then returning results.
2. Should service interact with other services?
#### 9. Controller: 
1. responsible for designing the APIs and setting the API specification so the front-end can interact with the back-end.
2. 

