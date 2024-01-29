---
title: SpringBoot — Spring Boot Batch Guide, Example with CSV To Database
date: 2024-01-29 00:00:00 Z
categories:
- SpringBoot
tags:
- SpringBoot
layout: article
cover: /assets/logo/springboot.png
sharing: true
license: false
aside:
  toc: true
pageview: true
---


Batch processing is a processing mode which involves execution of series of automated complex jobs without user interaction. A batch process handles bulk data and runs for a long time.

Spring Batch applications support −
-  Automatic retry after failure.
-  Tracking status and statistics during the batch execution and after completing the batch processing.
-  To run concurrent jobs.
-  Services such as logging, resource management, skip, and restarting the processing.



## Components of Spring Batch
 
![image](https://github.com/smlcodes/OnePageTutorials/assets/20472904/23926fba-469e-4196-a318-e9768eb666f8)

**1.Job**
In a Spring Batch application, a job is the batch process that is to be executed. It runs from start to finish without interruption. This job is further divided into steps (or a job contains steps).


**2.Step**
A step is an independent part of a job which contains the necessary information to define and execute the job (its part).
As specified in the diagram, each step is composed of an ItemReader, ItemProcessor (optional) and an ItemWriter. A job may contain one or more steps.


**3.Readers, Writers, and Processors**
-  Item reader reads data into a Spring Batch application from a particular source.
-  Item writer writes data from the Spring Batch application to a particular destination.
-  Item processor is a class which contains the processing code which processes the data read into the spring batch. If the application reads “n” records, then the code in the processor will be executed on each record.



For example, if we are writing a job with a simple step in it where we read data from CSV , process it and write it to a Database then our step uses 
-  A reader which reads from CSV file.
-  A writer which writes to Database.
-  A custom processor which processes the data as per our wish.



## Example 

1.Update Maven Dependencies
```java
<!-- Batch  -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```



2.Configure Spring Batch properties in application.yaml
```java
spring:
  batch:
    jdbc:
      initialize-schema: ALWAYS #To create spring_batch db tables
    job:
      enabled: false #To prevent automatic execution of Job at Startup
```



3.Spring Batch Configuration file.
This is main step in the batch processing. We will define SpringBatchConfig class with job details.
-  Job
-  ItemReader
-  ItemProcessor
-  ItemWriter


The Job method accepts following perameters. Here we will create Bean object for each parameter in separate method and pass it to Job. The parameters invoves here are
-  **JobBuilderFactory** provides a convenient way to create and configure instances of the Job class.
-  **StepBuilderFactory** provides methods to create and configure instances of the Step class.
-  **ItemReader** is used to read data from a source.
-  **ItemProcessor** takes UserDto objects as input and processes them, possibly transforming them into User objects. The processed items are then passed to the writer.
-  **ItemWriter** is responsible for taking the processed items (in this case, User objects) and writing them to a specified destination. This destination could be a database, a file, or any other type of output.


```java
@Configuration
@EnableBatchProcessing
public class SpringBatchConfig {
    @Bean
    public Job job(JobBuilderFactory jobBuilderFactory,
                   StepBuilderFactory stepBuilderFactory,
                   ItemReader<UserDto> itemReader,
                   ItemProcessor<UserDto, User> itemProcessor,
                   ItemWriter<User> itemWriter
    ) {

        Step step = stepBuilderFactory.get("ETL-file-load")
                .<UserDto, User>chunk(100)
                .reader(itemReader)
                .processor(itemProcessor)
                .writer(itemWriter)
                .build();


        return jobBuilderFactory.get("ETL-Load")
                .incrementer(new RunIdIncrementer())
                .start(step)
                .build();
    }


    @Bean
    @StepScope
    public FlatFileItemReader<UserDto> itemReader(@Value("#{jobParameters['inputFilePath']}") String inputFilePath) {
        FlatFileItemReader<UserDto> flatFileItemReader = new FlatFileItemReader<>();
        flatFileItemReader.setResource(new FileSystemResource(inputFilePath));
        flatFileItemReader.setName("CSV-Reader");
        flatFileItemReader.setLinesToSkip(1);
        flatFileItemReader.setLineMapper(lineMapper());
        return flatFileItemReader;
    }


    @Bean
    public LineMapper<UserDto> lineMapper() {
        //DefaultLineMapper:  is a class provided by the Spring Batch framework that is used for
        // mapping lines of input data to domain objects.
        DefaultLineMapper<UserDto> defaultLineMapper = new DefaultLineMapper<>();
        DelimitedLineTokenizer lineTokenizer = new DelimitedLineTokenizer();

        lineTokenizer.setDelimiter(",");
        lineTokenizer.setStrict(false);
        lineTokenizer.setNames("username", "password");

        //BeanWrapperFieldSetMapper  is used for mapping fields from a delimited or fixed-length file (typically a flat file)
        // to a Java object.
        BeanWrapperFieldSetMapper<UserDto> fieldSetMapper = new BeanWrapperFieldSetMapper<>();
        fieldSetMapper.setTargetType(UserDto.class);

        defaultLineMapper.setLineTokenizer(lineTokenizer);
        defaultLineMapper.setFieldSetMapper(fieldSetMapper);

        return defaultLineMapper;
    }

}
```


In above class we have defiend Job & ItemReader. Lets define ItemProcessor & ItemWriter in sperate classes.
```java
@Component
public class UserItemProcessor implements ItemProcessor<UserDto, User> {
    @Override
    public User process(UserDto userDto) throws Exception {
        User user = new User();
        user.setUsername(userDto.getUsername());
        user.setPassword(userDto.getPassword());
        return user;
    }
}
```



```java
@Component
public class UserItemWriter implements ItemWriter<User> {

    private UserRepository userRepository;

    @Autowired
    public UserItemWriter(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public void write(List<? extends User> users) throws Exception{
        System.out.println("Data Saved for Users: " + users);
        userRepository.saveAll(users);
    }
}
```


4. Other SpringBoot classes

//Controller
```java
@PostMapping("/upload")
public Object uploadFile(@RequestBody MultipartFile file, @RequestParam("isBatch") Boolean isBatch) {
    if (file.isEmpty()) {
        throw new BusinessException("File is Empty");
    }
 
        return userService.batchUploadUsers(file); 
}
```


//Service
```java
public interface UserService {

    BatchStatus batchUploadUsers(MultipartFile file);
}
```


```java
@Service
@Slf4j
@AllArgsConstructor
public class UserServiceImpl implements UserService {
    @Autowired
    private UserRepository userRepository;
    @Autowired
    private UserMapper userMapper;

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private Job job;


    @Override
    public BatchStatus batchUploadUsers(MultipartFile file) {
        try {
            // Save the uploaded file to a temporary location
            Path tempFile = Files.createTempFile("temp_users", ".csv");
            file.transferTo(tempFile.toFile());

            // Set the file path in the job parameters
            Map<String, JobParameter> maps = new HashMap<>();
            maps.put("timestamp", new JobParameter(System.currentTimeMillis()));
            maps.put("inputFilePath", new JobParameter(tempFile.toAbsolutePath().toString()));
            JobParameters parameters = new JobParameters(maps);

            // Run the job with the updated parameters
            JobExecution jobExecution = jobLauncher.run(job, parameters);

            // Clean up the temporary file (optional)
            Files.delete(tempFile);

            return jobExecution.getStatus();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```


Run the Application & Check the API
 <img width="468" alt="image" src="https://github.com/smlcodes/OnePageTutorials/assets/20472904/9091abb2-b89c-4eb0-86ac-342fc7cca3ea">


