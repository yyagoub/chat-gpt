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
```java
package sa.elm.iam2.portal.backend.module.domain.customer.profile;

import jakarta.persistence.*;
import lombok.Data;
import lombok.ToString;
import org.hibernate.annotations.Where;
import sa.elm.iam2.portal.backend.framework.jpa.model.EntityMeta;
import sa.elm.iam2.portal.backend.module.domain.customer.service.domain.CustomerServiceDomain;

import java.time.LocalDateTime;
import java.util.List;

@Entity
@Where(clause = "deleted = 0")
@Data
public class CustomerProfile extends EntityMeta {

	@Id
	@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "iam_customer_profile_generator")
	@SequenceGenerator(name="iam_customer_profile_generator", sequenceName = "iam_customer_profile_seq", allocationSize = 1)
	private Long id;
	private String nameAr;
	private String nameEn;
	private String city;
	private String phone;
	private String fax;
	private String email;
	private String commercialRegistrationCode;
	private LocalDateTime commercialRegistrationIssue;
	private LocalDateTime commercialRegistrationExpiry;
	private String vatNumber;
	private String POBox;
	private String POCode;
	private String website;

	private String sapNumber;

	@ToString.Exclude
	@OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.PERSIST)
	@JoinColumn(name = "customerProfileId")
	private List<CustomerServiceDomain> serviceDomains;

}
```

#### 2. DTO: 
1. represents the light form of the entity's data, which can be passed between services.
2. Naming of a DTO should have the Entity name with a suffex for example:
Entity name is CustomerProfile then DTO name is CustomerProfileDto
3. Lombok @Data used with every DTO 
4. @ToString.Exclude used with every complex object or DTO relation
5. extends DtoMeta in every entity to enhirt AuditingMetadata columns
```java
package sa.elm.iam2.portal.backend.module.domain.customer.profile;

import lombok.Data;
import lombok.ToString;
import sa.elm.iam2.portal.backend.framework.jpa.model.DtoMeta;
import sa.elm.iam2.portal.backend.module.domain.customer.service.domain.CustomerServiceDomainDto;

import java.time.LocalDateTime;
import java.util.List;

@Data
public class CustomerProfileDto extends DtoMeta {
	private Long id;
	private String nameAr;
	private String nameEn;
	private String city;
	private String phone;
	private String fax;
	private String email;
	private String commercialRegistrationCode;
	private LocalDateTime commercialRegistrationIssue;
	private LocalDateTime commercialRegistrationExpiry;
	private String vatNumber;
	private String POBox;
	private String POCode;
	private String website;

	private String sapNumber;

	@ToString.Exclude
	private List<CustomerServiceDomainDto> serviceDomains;

}

```

#### 3. Model: 
1. represents the client request; 
2. the main purpose of this is to control the data passed by the requester.
2. Naming of a Model should have the Entity name with a suffex for example:
Entity name is CustomerProfile then Model name is CustomerProfileModel
3. Lombok @Data used with every Model 
4. @ToString.Exclude used with every complex object or Model relation
5. extends ModelMeta in every entity to enhirt AuditingMetadata columns
```java
package sa.elm.iam2.portal.backend.module.domain.customer.profile;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Data;
import sa.elm.iam2.portal.backend.framework.jpa.model.ModelMeta;

import java.time.LocalDate;

@Data
public class CustomerProfileModel extends ModelMeta {

	private String nameAr;
	private String nameEn;
	private String city;
	private String phone;
	private String fax;
	private String email;
	private String commercialRegistrationCode;
	private LocalDate commercialRegistrationIssue;
	private LocalDate commercialRegistrationExpiry;
	private String vatNumber;
	@JsonProperty("pobox")
	private String POBox;
	@JsonProperty("pocode")
	private String POCode;
	private String website;

	private String sapNumber;
}
```

#### 4. Mapper: 
1. an interface that is stateful to all mapping happening within a domain
2. it may map the domain's entity or its joined entity.
3. Mapper is annotated with @Mapper 
4. extends interface StructMapper <E extends EntityMeta, M extends ModelMeta, D extends DtoMeta>
5. StructMap framework will implement the interfaces into classes.
6. Mapper must not map chiled/relation objects or lists.
7. if there is a complex objects it should be handled separatly to prevent stackoverslow.
8. there is no bean injection in a mapper class as there must not have a dependency ever.
9. Naming of a Mapper should have the Entity name with a suffex for example:
Entity name is CustomerProfile then Mapper name is CustomerProfileMapper
10. any data manipulation it must be within the Mapper.
```java
package sa.elm.iam2.portal.backend.module.domain.customer.profile;

import org.mapstruct.*;
import sa.elm.iam2.portal.backend.framework.mapping.StructMapper;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.List;

@Mapper
public interface CustomerProfileMapper extends StructMapper<CustomerProfile, CustomerProfileModel, CustomerProfileDto> {

    @Override
    @Named("entityToDto")
    @Mapping(target = "serviceDomains", ignore = true)
    CustomerProfileDto dto(final CustomerProfile entity);

    @Override
    @IterableMapping(qualifiedByName = "entityToDto")
    List<CustomerProfileDto> dto(final List<CustomerProfile> entity);

    @Override
    @Named("entity")
    @Mapping(target = "commercialRegistrationIssue", source = "model.commercialRegistrationIssue", qualifiedByName = "startOfDay")
    @Mapping(target = "commercialRegistrationExpiry", source = "model.commercialRegistrationExpiry", qualifiedByName = "startOfDay")
    @Mapping(target = "enabled", defaultValue = "false")
    CustomerProfile entity(final CustomerProfileModel model);

    @Override
    @IterableMapping(qualifiedByName = "entity")
    List<CustomerProfile> entity(final List<CustomerProfileModel> model);

    @Mapping(target = "createdByUsername", ignore = true)
    @Mapping(target = "lastModifiedByUsername", source = "createdByUsername")
    @Mapping(target = "commercialRegistrationIssue", source = "model.commercialRegistrationIssue", qualifiedByName = "startOfDay")
    @Mapping(target = "commercialRegistrationExpiry", source = "model.commercialRegistrationExpiry", qualifiedByName = "startOfDay")
    CustomerProfile entity(final CustomerProfileModel model, @MappingTarget final CustomerProfile profile);

    @Named("startOfDay")
    default LocalDateTime startOfDay(final LocalDate localDate) {
        return localDate.atStartOfDay();
    }

}
```

#### 5. Repository: 
1. All data access and manipulation should be written in this class.
2. write your query for readability.
3. extends ExtendedRepository<T extends EntityMeta, ID extends Serializable>
4. you should not use hard delete.
5. use soft delete method for delete: ExtendedRepository.delete(final Long id)
2. Naming of a Repository should have the Entity name with a suffex for example:
Entity name is CustomerProfile then Repository name is CustomerProfileRepository
```java
package sa.elm.iam2.portal.backend.module.domain.customer.profile;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.Query;
import sa.elm.iam2.portal.backend.framework.jpa.repository.ExtendedRepository;

import java.util.List;
import java.util.Optional;

public interface CustomerProfileRepository extends ExtendedRepository<CustomerProfile, Long> {

	boolean existsByCommercialRegistrationCode(final String commercialRegistrationCode);
	boolean existsByCommercialRegistrationCodeAndIdNot(final String commercialRegistrationCode, final Long id);
	@Query(" select c from CustomerProfile c order by c.id desc ")
	List<CustomerProfile> findAll();

	@Query("select cp from CustomerProfile cp " +
			"where cp.commercialRegistrationCode like %:search% " +
			"or cp.nameAr like %:search% " +
			"or cp.nameEn like %:search% " +
			"or cp.sapNumber like %:search% ")
    Page<CustomerProfile> findAll(Pageable pageable, String search);

	@Query("select cp from CustomerProfile cp " +
			"where cp.commercialRegistrationCode = :commercialRegistrationCode " )
	Optional<CustomerProfile> findByCommercialRegistrationCode(final String commercialRegistrationCode);

	@Query("select cp from CustomerProfile cp " +
			"where cp.commercialRegistrationCode = :commercialRegistrationCode " +
			"and cp.id <> :id ")
	Optional<CustomerProfile> findByCommercialRegistrationCodeAndNotId(final String commercialRegistrationCode, final Long id);
}
```

#### 6. Validator: 
1. All validation should be in this class; 
2. no other classes should throw an exception except for the Validator.
3. Validator must separate the model validation before any data access to prevent unnecessary data retreival. 
4. Validator throws only 422,400 errors and not 500.
2. Naming of a Validator should have the Entity name with a suffex for example:
Entity name is CustomerProfile then Validator name is CustomerProfileValidator
```java
package sa.elm.iam2.portal.backend.module.domain.customer.profile;

import org.springframework.stereotype.Service;
import sa.elm.iam2.portal.backend.config.exception.ErrorCode;

@Service
public class CustomerProfileValidator {

	public void validateBeforeRegisterProfile(final CustomerProfileModel customerProfileModel) {
		if (customerProfileModel.getCommercialRegistrationIssue().isAfter(customerProfileModel.getCommercialRegistrationExpiry()))
			throw new CustomerProfileUnprocessableEntityException(ErrorCode.IssueDateIsAfterExpiryDate);
		if (customerProfileModel.getCommercialRegistrationIssue() == null || customerProfileModel.getCommercialRegistrationExpiry() == null)
			throw new CustomerProfileUnprocessableEntityException(ErrorCode.DateNull);
	}

	public void beforeUpdate(final CustomerProfileModel customerProfileModel, final Long id) {
		if (customerProfileModel.getCommercialRegistrationExpiry() == null || customerProfileModel.getCommercialRegistrationIssue() == null)
			throw new CustomerProfileUnprocessableEntityException(ErrorCode.DateNull);
		if (customerProfileModel.getCommercialRegistrationIssue().isAfter(customerProfileModel.getCommercialRegistrationExpiry()))
			throw new CustomerProfileUnprocessableEntityException(ErrorCode.IssueDateIsAfterExpiryDate);
	}
}
```

#### 7. UnprocessableEntityException: 
1. represents any business validation error.
2. Naming of a UnprocessableEntityException should have the Entity name with a suffex for example:
Entity name is CustomerProfile then DTO name is CustomerProfileUnprocessableEntityException
3. extends UnprocessableEntityException
4. constructor must accept ErrorCode as parameter.
5. constructor must pass ExceptionCode to supper class.
6. each entity must define an enum on ExceptionCode class that will be statful to the entity.
```java
package sa.elm.iam2.portal.backend.module.domain.customer.profile;

import sa.elm.iam2.portal.backend.config.exception.ErrorCode;
import sa.elm.iam2.portal.backend.config.exception.ExceptionCode;
import sa.elm.iam2.portal.backend.framework.exception.type.UnprocessableEntityException;

public class CustomerProfileUnprocessableEntityException extends UnprocessableEntityException {
	public CustomerProfileUnprocessableEntityException(ErrorCode code) {
		super(ExceptionCode.CustomerProfile, code);
	}

}
```
#### 8. Service: 
1. service is presenting a story of how this service interacts with different behavior classes, such as validate request, then mapping to a DTO, then saving as an entity, then returning results.
2. Should service interact with other services?
``` java
package sa.elm.iam2.portal.backend.module.domain.customer.profile;

import jakarta.transaction.Transactional;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;
import sa.elm.iam2.portal.backend.config.exception.ErrorCode;
import org.springframework.data.domain.Page;

import java.util.List;
import java.util.Map;
import java.util.Optional;

@Service
@RequiredArgsConstructor
public class CustomerProfileService {

	private final CustomerProfileRepository repository;
	private final CustomerProfileMapper mapper;
	private final CustomerProfileValidator validator;

	@Transactional
	public CustomerProfileDto registerProfile(final CustomerProfileModel model) {
		this.validator.validateBeforeRegisterProfile(model);
		Optional<CustomerProfile> profile = this.repository.findByCommercialRegistrationCode(model.getCommercialRegistrationCode());
		if (profile.isPresent())
			throw new CustomerProfileUnprocessableEntityException(ErrorCode.ProfileCommRegExists);
		CustomerProfile customer = this.mapper.entity(model);
		return this.mapper.dto(this.repository.save(customer));
	}

	@Transactional
	public CustomerProfileDto updateProfile(final Long id, final CustomerProfileModel model) {
		this.validator.beforeUpdate(model, id);
		Optional<CustomerProfile> duplicatedProfile = this.repository.findByCommercialRegistrationCodeAndNotId(model.getCommercialRegistrationCode(), id);
		if (duplicatedProfile.isPresent())
			throw new CustomerProfileUnprocessableEntityException(ErrorCode.ProfileCommRegExists);
		CustomerProfile savedProfile = this.repository.findById(id).orElseThrow(() -> new CustomerProfileUnprocessableEntityException(ErrorCode.ProfileNotFound));
		CustomerProfile updatedProfile = this.mapper.entity(model, savedProfile);
		return this.mapper.dto(this.repository.save(updatedProfile));
	}

	public List<CustomerProfileDto> getAllProfiles() {
		return this.mapper.dto(this.repository.findAll());
	}

	public Page<CustomerProfileDto> getPage(final Map<String, String> params) {
		Pageable pageable = this.mapper.pageable(params);
		String search = this.mapper.search(params);
		if (StringUtils.hasText(search))
			return this.repository.findAll(pageable, search).map(this.mapper::dto);
		return this.repository.findAll(pageable).map(this.mapper::dto);
	}

	public CustomerProfileDto getProfileId(final Long id) {
		CustomerProfile customerProfile = this.repository.findById(id).orElseThrow(() -> new CustomerProfileUnprocessableEntityException(ErrorCode.ProfileNotFound));
		return this.mapper.dto(customerProfile);
	}
	
	@Transactional
	public void deleteProfile(final Long id) {
		CustomerProfile customerProfile = this.repository.findById(id).orElseThrow(() -> new CustomerProfileUnprocessableEntityException(ErrorCode.ProfileNotFound));
		this.repository.delete(customerProfile.getId());
	}

	public CustomerProfileDto enableProfile(final Long id) {
		CustomerProfile customerProfile = this.repository.findById(id).orElseThrow(() -> new CustomerProfileUnprocessableEntityException(ErrorCode.ProfileNotFound));
		customerProfile.setEnabled(!customerProfile.isEnabled());
		return this.mapper.dto(this.repository.save(customerProfile));
	}
}
```
#### 9. Controller: 
1. responsible for designing the APIs and setting the API specification so the front-end can interact with the back-end.
2. define the APIs authority level.
```java
package sa.elm.iam2.portal.backend.module.domain.customer.profile;

import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequestMapping("/api/v1/customer-profile")
@RequiredArgsConstructor
public class CustomerProfileController {

	private final CustomerProfileService customerProfileService;

	@GetMapping("{id}")
	public CustomerProfileDto getProfileById(@PathVariable final Long id) {
		return this.customerProfileService.getProfileId(id);
	}

	@GetMapping
	public Page<CustomerProfileDto> getAllProfiles(@RequestParam final Map<String, String> params) {
		return this.customerProfileService.getPage(params);
	}

	@PostMapping
	public CustomerProfileDto addProfile(@RequestBody final CustomerProfileModel customerProfileModel) {
		return this.customerProfileService.registerProfile(customerProfileModel);
	}

	@PutMapping("{id}")
	public CustomerProfileDto updateProfile(@PathVariable final Long id,@RequestBody final CustomerProfileModel customerProfileModel) {
		return this.customerProfileService.updateProfile(id, customerProfileModel);
	}

	@DeleteMapping("{id}")
	public void deleteProfile(@PathVariable("id") Long profileId) {
		this.customerProfileService.deleteProfile(profileId);
	}

	@PutMapping("enable/{id}")
	public CustomerProfileDto enableProfile(@PathVariable Long id) {
		return this.customerProfileService.enableProfile(id);
	}
}
```

