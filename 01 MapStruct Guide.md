
---
title: Java - MapStruct Guide
date: 2022-08-19 00:00:00 Z
categories:
- Java
tags:
- Java
layout: article
cover: /assets/logo/java.png
sharing: true
license: false
aside:
  toc: true
pageview: true
---


### MapStruct

MapStruct is an open-source Java-based code generator which creates code for mapping implementations.

It uses annotation-processing to generate mapper class implementations during compilation and greatly reduces the amount of boilerplate code which would regularly be written by hand.

### MapStruct Dependencies

If you're using Maven, install MapStruct by adding the dependency:

```
<dependencies>
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>${org.mapstruct.version}</version>
    </dependency>
</dependencies>

```

This dependency will import the core MapStruct annotations. Since MapStruct works on compile-time and is attached to builders like Maven and Gradle, we'll also have to add a plugin to the `<build>`:

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.5.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>${org.mapstruct.version}</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>

```

If you're using *Gradle*, installing MapStruct is as simple as:

```
plugins {
    id 'net.ltgt.apt' version '0.20'
}

apply plugin: 'net.ltgt.apt-idea'
apply plugin: 'net.ltgt.apt-eclipse'

dependencies {
    compile "org.mapstruct:mapstruct:${mapstructVersion}"
    annotationProcessor "org.mapstruct:mapstruct-processor:${mapstructVersion}"
}

```

The `net.ltgt.apt` plugin is responsible for the annotation processing. You can apply the `apt-idea` and `apt-eclipse` plugins depending on your IDE.

You can check out the latest version at [Maven Central](https://search.maven.org/search?q=g:org.mapstruct).

### Basic Mappings

Let's start off with some basic mapping. We'll have a `Doctor` model and a `DoctorDto`. Their fields will have the same names for our convenience:

```
public class Doctor {
    private int id;
    private String name;
}

```

And:

```
public class DoctorDto {
    private int id;
    private String name;
}

```

Now, to make a mapper between these two, we'll create a `DoctorMapper` interface. By annotating it with `@Mapper`, MapStruct knows that this is a mapper between our two classes:

```
@Mapper
public interface DoctorMapper {
    DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);
    DoctorDto toDto(Doctor doctor);
}

```

We have an `INSTANCE` of `DoctorMapper` type. This will be our "entry-point" to the instance once we generate the implementation.

AD

We've defined a `toDto()` method in the interface, which accepts a `Doctor` instance and returns a `DoctorDto` instance. This is enough for MapStruct to know that we'd like to map a `Doctor` instance to a `DoctorDto` instance.

When we build/compile the application, the MapStruct annotation processor plugin will pick up the `DoctorMapper` interface and generate an implementation for it:

```
public class DoctorMapperImpl implements DoctorMapper {
    @Override
    public DoctorDto toDto(Doctor doctor) {
        if ( doctor == null ) {
            return null;
        }
        DoctorDtoBuilder doctorDto = DoctorDto.builder();

        doctorDto.id(doctor.getId());
        doctorDto.name(doctor.getName());

        return doctorDto.build();
    }
}

```

The `DoctorMapperImpl` class now contains a `toDto()` method which maps our `Doctor` fields to the `DoctorDto` fields.

Now, to map a `Doctor` instance to a `DoctorDto` instance, we'd do:

```
DoctorDto doctorDto = DoctorMapper.INSTANCE.toDto(doctor);

```

Note: You might've noticed a `DoctorDtoBuilder` in the implementation above. We've omitted the implementation for brevity, since builders tend to be long. MapStruct will attempt to use your builder if it's present in the class. If not, it'll just instantiate it via the `new` keyword.

If you'd like to read more about the [Builder Design Pattern in Java](https://stackabuse.com/the-builder-design-pattern-in-java/), we've got you covered!

### Mappings Different Source and Target Fields

Oftentimes, a model and a DTO won't have the same field names. There can be slight variations due to team members assigning their own renditions, and how you'd like to pack the info for the service that called for the DTO.

MapStruct provides support to handle these situations via the `@Mapping` annotation.

#### Different Property Names

Let's update the `Doctor` class to include a `specialty`:

```
public class Doctor {
    private int id;
    private String name;
    private String specialty;
}

```

And for the `DoctorDto`, let's add a `specialization` field:

```
public class DoctorDto {
    private int id;
    private String name;
    private String specialization;
}

```

Now, we'll have to let our `DoctorMapper` know of this discrepancy. We'll do so by setting the `source` and `target` flags of the `@Mapping` annotation with both of these variants:

```
@Mapper
public interface DoctorMapper {
    DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

    @Mapping(source = "doctor.specialty", target = "specialization")
    DoctorDto toDto(Doctor doctor);
}

```

The `specialty` field of the `Doctor` class corresponds to the `specialization` field of the `DoctorDto` class.

After compiling the code, the annotation processor has generated this implementation:

```
public class DoctorMapperImpl implements DoctorMapper {
@Override
    public DoctorDto toDto(Doctor doctor) {
        if (doctor == null) {
            return null;
        }

        DoctorDtoBuilder doctorDto = DoctorDto.builder();

        doctorDto.specialization(doctor.getSpecialty());
        doctorDto.id(doctor.getId());
        doctorDto.name(doctor.getName());

        return doctorDto.build();
    }
}

```

#### Multiple Source Classes

Sometimes, a single class isn't enough to build a DTO. Sometimes, we want to aggregate values from multiple classes into a single DTO for the end user. This is also done by setting the appropriate flags in the `@Mapping` annotation:

Let's create another model `Education`:

```
public class Education {
    private String degreeName;
    private String institute;
    private Integer yearOfPassing;
}

```

And add a new field in `DoctorDto`:

```
public class DoctorDto {
    private int id;
    private String name;
    private String degree;
    private String specialization;
}

```

Now, let's update the `DoctorMapper` interface:

```
@Mapper
public interface DoctorMapper {
    DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

    @Mapping(source = "doctor.specialty", target = "specialization")
    @Mapping(source = "education.degreeName", target = "degree")
    DoctorDto toDto(Doctor doctor, Education education);
}

```

We've added another `@Mapping` annotation in which we've set the source as the `degreeName` of the `Education` class, and the `target` as the `degree` field of the `DoctorDto` class.

If the `Education` and `Doctor` classes contain fields with the same name - we'll have to let the mapper know which one to use or it'll throw an exception. If both models contain an `id`, we'll have to choose which `id` will be mapped to the DTO property.

### Mapping Child Entities

In most cases, POJO's don't contain *just* primitive data types. In most cases, they'll contain other classes. For example, a `Doctor` will have `1..n` patients:

```
public class Patient {
    private int id;
    private String name;
}

```

And let's make a `List` of them for the `Doctor`:

```
public class Doctor {
    private int id;
    private String name;
    private String specialty;
    private List<Patient> patientList;
}

```

Since `Patient` data will be transferred, we'll create a DTO for it too:

```
public class PatientDto {
    private int id;
    private String name;
}

```

And finally, let's update the `DoctorDto` with a `List` of the newly created `PatientDto`:

```
public class DoctorDto {
    private int id;
    private String name;
    private String degree;
    private String specialization;
    private List<PatientDto> patientDtoList;
}

```

AD

Before we change anything in the `DoctorMapper`, we'll have to make a mapper that converts between the `Patient` and `PatientDto` classes:

```
@Mapper
public interface PatientMapper {
    PatientMapper INSTANCE = Mappers.getMapper(PatientMapper.class);
    PatientDto toDto(Patient patient);
}

```

It's a basic mapper that just maps a couple of primitive data types.

Now, let's update our `DoctorMapper` to include the doctor's patients:

```
@Mapper(uses = {PatientMapper.class})
public interface DoctorMapper {

    DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

    @Mapping(source = "doctor.patientList", target = "patientDtoList")
    @Mapping(source = "doctor.specialty", target = "specialization")
    DoctorDto toDto(Doctor doctor);
}

```

Since we're working with another class that requires mapping, we've set the `uses` flag of the `@Mapper` annotation. This `@Mapper` uses another `@Mapper`. You can put as many classes/mappers here as you'd like - we only have one.

Because we've added this flag, when generating the mapper implementation for the `DoctorMapper` interface, MapStruct will also convert the `Patient` model into a `PatientDto` - since we've registered the `PatientMapper` for this task.

Now, compiling the application will result in a new implementation:

```
public class DoctorMapperImpl implements DoctorMapper {
    private final PatientMapper patientMapper = Mappers.getMapper( PatientMapper.class );

    @Override
    public DoctorDto toDto(Doctor doctor) {
        if ( doctor == null ) {
            return null;
        }

        DoctorDtoBuilder doctorDto = DoctorDto.builder();

        doctorDto.patientDtoList( patientListToPatientDtoList(doctor.getPatientList()));
        doctorDto.specialization( doctor.getSpecialty() );
        doctorDto.id( doctor.getId() );
        doctorDto.name( doctor.getName() );

        return doctorDto.build();
    }

    protected List<PatientDto> patientListToPatientDtoList(List<Patient> list) {
        if ( list == null ) {
            return null;
        }

        List<PatientDto> list1 = new ArrayList<PatientDto>( list.size() );
        for ( Patient patient : list ) {
            list1.add( patientMapper.toDto( patient ) );
        }

        return list1;
    }
}

```

Evidently, a new mapper - `patientListToPatientDtoList()` has been added, besides the `toDto()` mapper. This is done without explicit definition, simply because we've added the `PatientMapper` to the `DoctorMapper`.

The method iterates over a list of `Patient` models, converts them to `PatientDto`s and adds them to a list contained within a `DoctorDto` object.

### Updating Existing Instances

Sometimes, we'd wish to update a model with the latest values from a DTO. Using the `@MappingTarget` annotation on the target object (`Doctor` in our case), we can update existing instances.

Let's add a new `@Mapping` to our `DoctorMapper` which accepts `Doctor` and `DoctorDto` instances. The `DoctorDto` instance will be the data source, while the `Doctor` will be the target:

```
@Mapper(uses = {PatientMapper.class})
public interface DoctorMapper {

    DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

    @Mapping(source = "doctorDto.patientDtoList", target = "patientList")
    @Mapping(source = "doctorDto.specialization", target = "specialty")
    void updateModel(DoctorDto doctorDto, @MappingTarget Doctor doctor);
}

```

Now, after generating the implementation again, we've got the `updateModel()` method:

```
public class DoctorMapperImpl implements DoctorMapper {

    @Override
    public void updateModel(DoctorDto doctorDto, Doctor doctor) {
        if (doctorDto == null) {
            return;
        }

        if (doctor.getPatientList() != null) {
            List<Patient> list = patientDtoListToPatientList(doctorDto.getPatientDtoList());
            if (list != null) {
                doctor.getPatientList().clear();
                doctor.getPatientList().addAll(list);
            }
            else {
                doctor.setPatientList(null);
            }
        }
        else {
            List<Patient> list = patientDtoListToPatientList(doctorDto.getPatientDtoList());
            if (list != null) {
                doctor.setPatientList(list);
            }
        }
        doctor.setSpecialty(doctorDto.getSpecialization());
        doctor.setId(doctorDto.getId());
        doctor.setName(doctorDto.getName());
    }
}

```

What's worth noting is that the patient list is also getting updated, since it's a child entity of the module.

### Dependency Injection

So far, we've been accessing the generated mappers via the `getMapper()` method:

```
DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

```

However, if you're using [Spring](https://spring.io/projects/spring-framework), you can update your mapper configuration and inject it like a regular dependency.

Let's update our `DoctorMapper` to work with Spring:

```
@Mapper(componentModel = "spring")
public interface DoctorMapper {}

```

Adding `(componentModel = "spring")` in the `@Mapper` annotation tells MapStruct that when generating the mapper implementation class, we'd like it to be created with the dependency injection support via Spring. Now, there's no need to add the `INSTANCE` field to the interface.

The generated `DoctorMapperImpl` will now have the `@Component` annotation:

```
@Component
public class DoctorMapperImpl implements DoctorMapper {}

```

Once marked as a `@Component`, Spring can pick it up as a bean and you're free to `@Autowire` it in another class such as a controller:

```
@Controller
public class DoctorController() {
    @Autowired
    private DoctorMapper doctorMapper;
}

```

If you're not using Spring, MapStruct has support for [Java CDI](https://docs.oracle.com/javaee/6/tutorial/doc/giwhl.html) as well:

```
@Mapper(componentModel = "cdi")
public interface DoctorMapper {}

```

### Mapping Enums

Mapping Enums works in the same way as mapping fields does. MapStruct will map the ones with the same names without a problem. Though, for Enums with different names, we'll be using the `@ValueMapping` annotation. Again, this is similar to the `@Mapping` annotation with regular types.

Let's create two Enums, the first one being `PaymentType`:

```
public enum PaymentType {
    CASH,
    CHEQUE,
    CARD_VISA,
    CARD_MASTER,
    CARD_CREDIT
}

```

This are, say, the available options for payment in an application. And now, let's have a more general, limited view of those options:

```
public enum PaymentTypeView {
    CASH,
    CHEQUE,
    CARD
}

```

Now, let's make a mapper interface between these two `enum`s:

![](https://s3.stackabuse.com/media/ebooks/git-essentials/git-essentials-cover-transparent-cropped.png)

Free eBook: Git Essentials
--------------------------

Check out our hands-on, practical guide to learning Git, with best-practices, industry-accepted standards, and included cheat sheet. Stop Googling Git commands and actually *learn* it!

Download the eBook  

```
@Mapper
public interface PaymentTypeMapper {

    PaymentTypeMapper INSTANCE = Mappers.getMapper(PaymentTypeMapper.class);

    @ValueMappings({
            @ValueMapping(source = "CARD_VISA", target = "CARD"),
            @ValueMapping(source = "CARD_MASTER", target = "CARD"),
            @ValueMapping(source = "CARD_CREDIT", target = "CARD")
    })
    PaymentTypeView paymentTypeToPaymentTypeView(PaymentType paymentType);
}

```

Here, we've got a general `CARD` value, and more specific `CARD_VISA`, `CARD_MASTER` and `CARD_CREDIT` values. There's a mismatch with the number of values - `PaymentType` has 6 values, whereas `PaymentTypeView` only has 3.

To bridge between these, we can use the `@ValueMappings` annotation, which accepts multiple `@ValueMapping` annotations. Here, we can set the source to be any of the three specific cases, and the target as the `CARD` value.

MapStruct will handle these cases:

```
public class PaymentTypeMapperImpl implements PaymentTypeMapper {

    @Override
    public PaymentTypeView paymentTypeToPaymentTypeView(PaymentType paymentType) {
        if (paymentType == null) {
            return null;
        }

        PaymentTypeView paymentTypeView;

        switch (paymentType) {
            case CARD_VISA: paymentTypeView = PaymentTypeView.CARD;
            break;
            case CARD_MASTER: paymentTypeView = PaymentTypeView.CARD;
            break;
            case CARD_CREDIT: paymentTypeView = PaymentTypeView.CARD;
            break;
            case CASH: paymentTypeView = PaymentTypeView.CASH;
            break;
            case CHEQUE: paymentTypeView = PaymentTypeView.CHEQUE;
            break;
            default: throw new IllegalArgumentException( "Unexpected enum constant: " + paymentType );
        }
        return paymentTypeView;
    }
}

```

`CASH` and `CHEQUE` have their corresponding values by default, whereas the specific `CARD` value is handled through a `switch` loop.

Though, this approach can become impractical when you have a lot of values you'd like to assign to a more general one. Instead of assigning each one manually, we can simply let MapStruct go through all of the available remaining values and map them all to another one.

This is done via `MappingConstants`:

```
@ValueMapping(source = MappingConstants.ANY_REMAINING, target = "CARD")
PaymentTypeView paymentTypeToPaymentTypeView(PaymentType paymentType);

```

Here, after the default mappings are done, any remaining (not matching) values will all be mapped to `CARD`.

```
@Override
public PaymentTypeView paymentTypeToPaymentTypeView(PaymentType paymentType) {
    if ( paymentType == null ) {
        return null;
    }

    PaymentTypeView paymentTypeView;

    switch ( paymentType ) {
        case CASH: paymentTypeView = PaymentTypeView.CASH;
        break;
        case CHEQUE: paymentTypeView = PaymentTypeView.CHEQUE;
        break;
        default: paymentTypeView = PaymentTypeView.CARD;
    }
    return paymentTypeView;
}

```

Another option would be to use `ANY_UNMAPPED`:

```
@ValueMapping(source = MappingConstants.ANY_UNMAPPED, target = "CARD")
PaymentTypeView paymentTypeToPaymentTypeView(PaymentType paymentType);

```

In this case, instead of mapping default values first, followed with mapping the remaining ones to a single target - MapStruct will just map *all* unmapped values to the target.

### Mapping DataTypes

MapStruct supports data type conversion between `source` and `target` properties. It also provides automatic type conversion between primitive types and their corresponding wrappers.

Automatic type conversion applies to:

-   Conversion between *primitive types* and their *respective wrapper types*. For example, conversion between `int` and `Integer`, `float` and `Float`, `long` and `Long`, `boolean` and `Boolean` etc.
-   Conversion between *any primitive types* and *any wrapper types*. For example, between `int` and `long`, `byte` and `Integer` etc.
-   Conversion between all *primitive and wrapper types* and `String`. For example, conversion between `boolean` and `String`, `Integer` and `String`, `float` and `String` etc.

So during mapper code generation if the type conversion between source and target field falls under any of the above scenarios, MapStrcut will handle the type conversion itself.

Let update our `PatientDto` to include a field for storing the `dateofBirth`:

```
public class PatientDto {
    private int id;
    private String name;
    private LocalDate dateOfBirth;
}

```

On the other hand, say our `Patient` object has a `dateOfBirth` of type `String`:

```
public class Patient {
    private int id;
    private String name;
    private String dateOfBirth;
}

```

Now, let's go ahead and make a mapper between these two:

```
@Mapper
public interface PatientMapper {

    @Mapping(source = "dateOfBirth", target = "dateOfBirth", dateFormat = "dd/MMM/yyyy")
    Patient toModel(PatientDto patientDto);
}

```

When converting between dates, we can also use the `dateFormat` flag to set the format specifier. The generated implementation will look like:

```
public class PatientMapperImpl implements PatientMapper {

    @Override
    public Patient toModel(PatientDto patientDto) {
        if (patientDto == null) {
            return null;
        }

        PatientBuilder patient = Patient.builder();

        if (patientDto.getDateOfBirth() != null) {
            patient.dateOfBirth(DateTimeFormatter.ofPattern("dd/MMM/yyyy")
                                .format(patientDto.getDateOfBirth()));
        }
        patient.id(patientDto.getId());
        patient.name(patientDto.getName());

        return patient.build();
    }
}

```

Note that MapStruct has used the pattern provided by the `dateFormat` flag. If we didn't specify the format, it would've been set to the default format of a `LocalDate`:

```
if (patientDto.getDateOfBirth() != null) {
    patient.dateOfBirth(DateTimeFormatter.ISO_LOCAL_DATE
                        .format(patientDto.getDateOfBirth()));
}

```

### Adding Custom Methods

So far, we've been adding a placeholder method that we want MapStruct to implement for us. What we can also do is add a custom `default` method to the interface as well. By adding a `default` method, we can add the implementation directly as well. We'll be able to access it through the instance without a problem.

For this, let's make a `DoctorPatientSummary`, which contains a summary between a `Doctor` and a list of their `Patient`s:

```
public class DoctorPatientSummary {
    private int doctorId;
    private int patientCount;
    private String doctorName;
    private String specialization;
    private String institute;
    private List<Integer> patientIds;
}

```

Now, in our `DoctorMapper`, we'll add a `default` method which, instead of mapping a `Doctor` to a `DoctorDto`, converts the `Doctor` and `Education` objects into a `DoctorPatientSummary`:

```
@Mapper
public interface DoctorMapper {

    default DoctorPatientSummary toDoctorPatientSummary(Doctor doctor, Education education) {

        return DoctorPatientSummary.builder()
                .doctorId(doctor.getId())
                .doctorName(doctor.getName())
                .patientCount(doctor.getPatientList().size())
				.patientIds(doctor.getPatientList()
            	        .stream()
                        .map(Patient::getId)
            	        .collect(Collectors.toList()))
            	.institute(education.getInstitute())
                .specialization(education.getDegreeName())
                .build();
    }
}

```

This object is built from the `Doctor` and `Education` objects using the Builder Design pattern.

AD

This implementation will be available to use after the mapper implementation class is generated by MapStruct. You can access it just as you'd access any other mapper method:

```
DoctorPatientSummary summary = doctorMapper.toDoctorPatientSummary(dotor, education);

```

### Creating Custom Mappers

So far, we've been using interfaces to create blueprints for mappers. We can also make blueprints with `abstract` classes, annotated with the `@Mapper` annotation. MapStruct will create an implementation for this class, similar to creating an interface implementation.

Let's rewrite the previous example, though this time, we'll make it an `abstract` class:

```
@Mapper
public abstract class DoctorCustomMapper {
    public DoctorPatientSummary toDoctorPatientSummary(Doctor doctor, Education education) {

        return DoctorPatientSummary.builder()
                .doctorId(doctor.getId())
                .doctorName(doctor.getName())
                .patientCount(doctor.getPatientList().size())
                .patientIds(doctor.getPatientList()
                        .stream()
                        .map(Patient::getId)
                        .collect(Collectors.toList()))
                .institute(education.getInstitute())
                .specialization(education.getDegreeName())
                .build();
    }
}

```

You can use this implementation the same way as you'd use an interface implementation. Using `abstract` classes gives us more control and options when creating custom implementations due to less limitations. Another benefit is the ability to add `@BeforeMapping` and `@AfterMapping` methods.

### @BeforeMapping and @AfterMapping

For additional control and customization, we can define `@BeforeMapping` and `@AfterMapping` methods. Obviously, these run before and after each mapping. That is to say, these methods will be added and executed before and after the actual mapping between two objects within the implementation.

Let's add these methods to our `DoctorCustomMapper`:

```
@Mapper(uses = {PatientMapper.class}, componentModel = "spring")
public abstract class DoctorCustomMapper {

    @BeforeMapping
    protected void validate(Doctor doctor) {
        if(doctor.getPatientList() == null){
            doctor.setPatientList(new ArrayList<>());
        }
    }

    @AfterMapping
    protected void updateResult(@MappingTarget DoctorDto doctorDto) {
        doctorDto.setName(doctorDto.getName().toUpperCase());
        doctorDto.setDegree(doctorDto.getDegree().toUpperCase());
        doctorDto.setSpecialization(doctorDto.getSpecialization().toUpperCase());
    }

    @Mapping(source = "doctor.patientList", target = "patientDtoList")
    @Mapping(source = "doctor.specialty", target = "specialization")
    public abstract DoctorDto toDoctorDto(Doctor doctor);
}

```

Now, let's generate a mapper based on this class:

```
@Component
public class DoctorCustomMapperImpl extends DoctorCustomMapper {

    @Autowired
    private PatientMapper patientMapper;

    @Override
    public DoctorDto toDoctorDto(Doctor doctor) {
        validate(doctor);

        if (doctor == null) {
            return null;
        }

        DoctorDto doctorDto = new DoctorDto();

        doctorDto.setPatientDtoList(patientListToPatientDtoList(doctor
            .getPatientList()));
        doctorDto.setSpecialization(doctor.getSpecialty());
        doctorDto.setId(doctor.getId());
        doctorDto.setName(doctor.getName());

        updateResult(doctorDto);

        return doctorDto;
    }
}

```

The `validate()` method is ran before the `DoctorDto` object is instantiated, and the `updateResult()` method is ran after the mapping has finished.

### Adding Default Values

A useful couple of flags you can use with the `@Mapping` annotation are constants and default values. A `constant` value will always be used, regardless of the `source`'s value. A `default` value will be used if the `source` value is `null`.

Let's update our `DoctorMapper` with a `constant` and `default`:

```
@Mapper(uses = {PatientMapper.class}, componentModel = "spring")
public interface DoctorMapper {
    @Mapping(target = "id", constant = "-1")
    @Mapping(source = "doctor.patientList", target = "patientDtoList")
    @Mapping(source = "doctor.specialty", target = "specialization", defaultValue = "Information Not Available")
    DoctorDto toDto(Doctor doctor);
}

```

If the specialty isn't available, we'll assign the `Information Not Available` string instead. Also, we've hardcoded the `id` to be `-1`.

Let's generate the mapper:

```
@Component
public class DoctorMapperImpl implements DoctorMapper {

    @Autowired
    private PatientMapper patientMapper;

    @Override
    public DoctorDto toDto(Doctor doctor) {
        if (doctor == null) {
            return null;
        }

        DoctorDto doctorDto = new DoctorDto();

        if (doctor.getSpecialty() != null) {
            doctorDto.setSpecialization(doctor.getSpecialty());
        }
        else {
            doctorDto.setSpecialization("Information Not Available");
        }
        doctorDto.setPatientDtoList(patientListToPatientDtoList(doctor.getPatientList()));
        doctorDto.setName(doctor.getName());

        doctorDto.setId(-1);

        return doctorDto;
    }
}

```

If `doctor.getSpecialty()` returns `null`, we set the specialization to our default message. The `id` is set regardless, since it's a `constant`.

### Adding Java Expressions

MapStruct goes as far as allowing you to fully input Java expressions as flags to the `@Mapping` annotation. You can either set a `defaultExpression` (if the `source` value is `null`) or an `expression` which is constant.

Let's add an `externalId` which will be a `String` and an `appointment` which will be of `LocalDateTime` type to our `Doctor` and `DoctorDto`.

Our `Doctor` model will look like:

```
public class Doctor {

    private int id;
    private String name;
    private String externalId;
    private String specialty;
    private LocalDateTime availability;
    private List<Patient> patientList;
}

```

And `DoctorDto` will look like:

```
public class DoctorDto {

    private int id;
    private String name;
    private String externalId;
    private String specialization;
    private LocalDateTime availability;
    private List<PatientDto> patientDtoList;
}

```

And now, let's update our `DoctorMapper`:

```
@Mapper(uses = {PatientMapper.class}, componentModel = "spring", imports = {LocalDateTime.class, UUID.class})
public interface DoctorMapper {

    @Mapping(target = "externalId", expression = "java(UUID.randomUUID().toString())")
    @Mapping(source = "doctor.availability", target = "availability", defaultExpression = "java(LocalDateTime.now())")
    @Mapping(source = "doctor.patientList", target = "patientDtoList")
    @Mapping(source = "doctor.specialty", target = "specialization")
    DoctorDto toDtoWithExpression(Doctor doctor);
}

```

Here, we've assigned the value of `java(UUID.randomUUID().toString())` to the `externalId`, while we've conditionally set the availability to a new `LocalDateTime`, if the `availability` isn't present.

Since the expressions are just `String`s, we have to specify the classes used in the expressions. This isn't code that's being evaluated, it's a literal text value. Thus, we've added `imports = {LocalDateTime.class, UUID.class}` to the `@Mapper` annotation.

The generated mapper will look like:

```
@Component
public class DoctorMapperImpl implements DoctorMapper {

    @Autowired
    private PatientMapper patientMapper;

    @Override
    public DoctorDto toDtoWithExpression(Doctor doctor) {
        if (doctor == null) {
            return null;
        }

        DoctorDto doctorDto = new DoctorDto();

        doctorDto.setSpecialization(doctor.getSpecialty());
        if (doctor.getAvailability() != null) {
            doctorDto.setAvailability(doctor.getAvailability());
        }
        else {
            doctorDto.setAvailability(LocalDateTime.now());
        }
        doctorDto.setPatientDtoList(patientListToPatientDtoList(doctor
            .getPatientList()));
        doctorDto.setId(doctor.getId());
        doctorDto.setName(doctor.getName());

        doctorDto.setExternalId(UUID.randomUUID().toString());

        return doctorDto;
    }
}

```

The `externalId` is set to:

```
doctorDto.setExternalId(UUID.randomUUID().toString());

```

Whereas, if the `availability` is `null`, it's set to:

```
doctorDto.setAvailability(LocalDateTime.now());

```

### Exception Handling while Mapping

Exception Handling is unavoidable. Applications incur exceptional states all the time. MapStruct provides support to include exception handling pretty seamlessly, making your job as a dev a lot simpler.

Let's consider a scenario where we want to validate our `Doctor` model while mapping it to `DoctorDto`. Let's make a separate `Validator` class for this:

AD

```
public class Validator {
    public int validateId(int id) throws ValidationException {
        if(id == -1){
            throw new ValidationException("Invalid value in ID");
        }
        return id;
    }
}

```

Now, we'll want to update our `DoctorMapper` to use the `Validator` class, without us having to specify the implementation. As usual, we'll add the classes to the list of classes used by `@Mapper`, and all we have to do is tell MapStruct that our `toDto()` method `throws ValidationException`:

```
@Mapper(uses = {PatientMapper.class, Validator.class}, componentModel = "spring")
public interface DoctorMapper {

    @Mapping(source = "doctor.patientList", target = "patientDtoList")
    @Mapping(source = "doctor.specialty", target = "specialization")
    DoctorDto toDto(Doctor doctor) throws ValidationException;
}

```

Now, let's generate an implementation for this mapper:

```
@Component
public class DoctorMapperImpl implements DoctorMapper {

    @Autowired
    private PatientMapper patientMapper;
    @Autowired
    private Validator validator;

    @Override
    public DoctorDto toDto(Doctor doctor) throws ValidationException {
        if (doctor == null) {
            return null;
        }

        DoctorDto doctorDto = new DoctorDto();

        doctorDto.setPatientDtoList(patientListToPatientDtoList(doctor
            .getPatientList()));
        doctorDto.setSpecialization(doctor.getSpecialty());
        doctorDto.setId(validator.validateId(doctor.getId()));
        doctorDto.setName(doctor.getName());
        doctorDto.setExternalId(doctor.getExternalId());
        doctorDto.setAvailability(doctor.getAvailability());

        return doctorDto;
    }
}

```

MapStruct has automatically set the id of `doctorDto` with the result of the `Validator` instance. It also added a `throws` clause for the method.

### Mapping Configurations

MapStruct provides some very helpful configuration for writing mapper methods. Most of the time, the mapping configurations we specify for a mapper method are replicated when adding another mapper method for similar types.

Instead of configuring these manually, we can configure similar types to have the same/similar mapping methods.

#### Inherit Configuration

Let's revisit the scenario in [Updating Existing Instances](https://stackabuse.com/guide-to-mapstruct-in-java-advanced-mapping-library/#updatingexistingintances), where we created a mapper to update the values of an existing `Doctor` model from a `DoctorDto` object:

```
@Mapper(uses = {PatientMapper.class})
public interface DoctorMapper {

    DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

    @Mapping(source = "doctorDto.patientDtoList", target = "patientList")
    @Mapping(source = "doctorDto.specialization", target = "specialty")
    void updateModel(DoctorDto doctorDto, @MappingTarget Doctor doctor);
}

```

Say we have another mapper that generates a `Doctor` from a `DoctorDto`:

```
@Mapper(uses = {PatientMapper.class, Validator.class})
public interface DoctorMapper {

    @Mapping(source = "doctorDto.patientDtoList", target = "patientList")
    @Mapping(source = "doctorDto.specialization", target = "specialty")
    Doctor toModel(DoctorDto doctorDto);
}

```

Both these mapper methods use the same configuration. The `source`s and `target`s are the same. Instead of repeating the configurations for both mappers methods, we can use the `@InheritConfiguration` annotation.

By annotating a method with the `@InheritConfiguration` annotation, MapStruct will look for another, already configured method whose configuration can be applied to this one as well. Typically, this annotation is used for update methods after a mapping method, just like we're using it:

```
@Mapper(uses = {PatientMapper.class, Validator.class}, componentModel = "spring")
public interface DoctorMapper {

    @Mapping(source = "doctorDto.specialization", target = "specialty")
    @Mapping(source = "doctorDto.patientDtoList", target = "patientList")
    Doctor toModel(DoctorDto doctorDto);

    @InheritConfiguration
    void updateModel(DoctorDto doctorDto, @MappingTarget Doctor doctor);
}

```

#### Inherit Inverse Configuration

Another similar scenario is writing mapper functions to map *Model* to *DTO* and *DTO* to *Model*, like in the code below were we have to specify same source target mapping on both functions:

Your configurations won't always be *the same*. For example, they can be inverse. Mapping a model to a DTO and a DTO to a model - you use the same fields, but inverse. Here's how it looks like typically:

```
@Mapper(componentModel = "spring")
public interface PatientMapper {

    @Mapping(source = "dateOfBirth", target = "dateOfBirth", dateFormat = "dd/MMM/yyyy")
    Patient toModel(PatientDto patientDto);

    @Mapping(source = "dateOfBirth", target = "dateOfBirth", dateFormat = "dd/MMM/yyyy")
    PatientDto toDto(Patient patient);
}

```

Instead of writing this two times, we can use the `@InheritInverseConfiguration` annotation on the second method:

```
@Mapper(componentModel = "spring")
public interface PatientMapper {

    @Mapping(source = "dateOfBirth", target = "dateOfBirth", dateFormat = "dd/MMM/yyyy")
    Patient toModel(PatientDto patientDto);

    @InheritInverseConfiguration
    PatientDto toDto(Patient patient);
}

```

The generated code from both mapper implementations will be the same.

### Conclusion


Ref.

Covers All 
https://stackabuse.com/guide-to-mapstruct-in-java-advanced-mapping-library/

https://www.youtube.com/watch?v=ICl9gJ4o7Ec&ab_channel=Devoxx
https://www.youtube.com/watch?v=vHIiKB4RL2Y&ab_channel=CodewithB

