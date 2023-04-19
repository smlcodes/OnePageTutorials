Spring Boot supports several authentication types to secure your application. Here are the most commonly used authentication types:

1.  **Basic Authentication**: Basic authentication is a simple authentication scheme built into the HTTP protocol. With this type of authentication, the user's credentials are sent in the header of each request.

2.  **OAuth2**: OAuth2 is an open standard for authorization that allows users to grant third-party applications access to their resources without giving them their credentials. Spring Boot provides OAuth2 support using the Spring Security OAuth2 module.

3. **JWT Authentication** (JSON Web Tokens ): JSON Web Tokens are a secure way to transmit information between parties as a JSON object. JWTs are commonly used for authentication and authorization purposes in modern web applications.

4.  **LDAP Authentication**: LDAP authentication allows Spring Boot to authenticate users against an LDAP directory service. This is useful for enterprise applications that already have an LDAP infrastructure in place.

5.  **SAML Authentication**: SAML (Security Assertion Markup Language) is an XML-based standard for exchanging authentication and authorization data between parties. Spring Boot provides support for SAML-based authentication using the Spring Security SAML module.

These are just some of the authentication types supported by Spring Boot. Depending on the requirements of your application, you can choose the appropriate authentication type or even combine multiple types to provide a more secure and flexible authentication mechanism.



# How Spring Security Works
![image](https://user-images.githubusercontent.com/20472904/233063646-d0f3ceac-d894-4c62-93ed-5c658a2b20e6.png)

1.	When ever user request Rest API with username & password, request goes to Authentication Filter

2.	The **AuthenticationFilter** is a servlet filter class that will see if the user has already authenticated or not. If not, it will send that request to the **AuthenticationManager** to check if the details sent by the user is valid or not.

3.	If the username and password are valid, the **AuthenticationManager** in turn uses **AuthenticationProvider** where the login logic or the authentication logic is defined.

4.	The Authentication Provider will not fetch the user details from the database or from LDAP or in memory. It will use UserDetailsService & PasswordEncoder for that purpose.


5.	Once the **AuthenticationProvider** checks, if the authentication details, the username, password, etc, are correct, then it will send the appropriate response back to the AuthenticationManager. AuthenticationManager hands it back to the AuthenticationFilter.

6.	If the user details are correct & authentication succeeded.
The AuthenticationFilter will use a AuthenticationSuccessHandler and stores that authentication information in  **SecurityContext**. 


7.	If the authentication failed, it will use the AuthenticationFailureHandler to send the appropriate response back to the client.



#### SecurityContext VS JSESSIONID

   For the first time the authentication happens, the JSESSIONID is created & stores inside of the SecurityContext as JSessionID as key & all the authentication information as value.

   from next time as the client sends in the JSESSIONID, AuthenticationFilter uses that JSESSIONID checks if he's already authenticated. And it will not prompt for authentication as long as that cookie is available. And when you deleted it, it will ask for the authentication again.

   Once Authenication is success AuthenticationFilter will return JSESSIONID cookie back to Browser/Postman. If now change the password, it will work because it has valid JSESSIONID in the cookie. If we remove JSESSIONID from postman/browser it shows 401-UnAuthorized. We need to provide valid creds again
 






# Basic Authentication

Basic Authentication is a simple authentication scheme that sends the user's credentials in the header of each request.  

Basic authentification is a standard HTTP header with the user and password encoded in base64 : `Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==`.The userName and password is encoded in the format `username:password`. This is one of the simplest technique to protect the REST resources because it does not require cookies. session identifiers or any login pages.

**1. Maven Dependency**. 

The simplest way to add all required jars is to add the *latest version of [spring-boot-starter-security](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security)* dependency.

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```


**2. Configure Spring Security**.  

To enable authentication and authorization support, we can configure the utility class *WebSecurityConfigurerAdapter* (*deprecated*). It helps in requiring the user to be authenticated prior to accessing any configured URL (or all URLs) within our application.

In the following configuration, we are using *httpBasic()* which enables basic authentication. We are also configuring the in-memory authentication manager to supply a username and password.

SecurityConfig.java // DEPRECATED 2.7.0
```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter
{
    @Override
    protected void configure(HttpSecurity http) throws Exception
    {
        http
         .csrf().disable()
         .authorizeRequests().anyRequest().authenticated()
         .and()
         .httpBasic();
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth)
            throws Exception
    {
        auth.inMemoryAuthentication()
        	.withUser("admin")
        	.password("{noop}password")
        	.roles("ROLE_USER");
    }
}
```

Starting Spring Boot 2.7.0, *WebSecurityConfigurerAdapter* is deprecated. We can rewrite the above basic auth configuration in the latest versions as follows:

```
@Configuration
public class BasicAuthWebSecurityConfiguration
{
  @Autowired
  private AppBasicAuthenticationEntryPoint authenticationEntryPoint;

  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/public").permitAll()
        .anyRequest().authenticated()
        .and()
        .httpBasic()
        .authenticationEntryPoint(authenticationEntryPoint);
    return http.build();
  }

  @Bean
  public InMemoryUserDetailsManager userDetailsService() {
    UserDetails user = User
        .withUsername("user")
        .password(passwordEncoder().encode("password"))
        .roles("USER_ROLE")
        .build();
    return new InMemoryUserDetailsManager(user);
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(8);
  }
}
```


**BasicAuthenticationEntryPoint**

The authentication entry points are used by the `ExceptionTranslationFilter` to commence authentication. By default, the *BasicAuthenticationEntryPoint* returns a full page for a *401 Unauthorized* response back to the client.

To customize the default authentication error page used by basic auth, we can extend the *BasicAuthenticationEntryPoint* class. Here we can set the [realm name](https://docs.oracle.com/cd/E19798-01/821-1841/6nmq2cpjd/index.html) and well as the error message sent back to the client.

```
@Component
public class AppBasicAuthenticationEntryPoint
  extends BasicAuthenticationEntryPoint {

  @Override
  public void commence(HttpServletRequest request,
                       HttpServletResponse response,
                       AuthenticationException authEx) throws IOException {

    response.addHeader("WWW-Authenticate", "Basic realm=" + getRealmName() + "");
    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
    PrintWriter writer = response.getWriter();
    writer.println("HTTP Status 401 - " + authEx.getMessage());
  }

  @Override
  public void afterPropertiesSet() {
    setRealmName("howtodoinjava");
    super.afterPropertiesSet();
  }
}
```

**3.Basic Authentication Demo**. 

For demo purposes, we can write a simple REST API given below.

EmployeeController.java

```
@RestController
@RequestMapping(path = "/employees")
public class EmployeeController
{
    @Autowired
    private EmployeeDAO employeeDao;

    @GetMapping(path="/", produces = "application/json")
    public Employees getEmployees()
    {
        return employeeDao.getAllEmployees();
    }
}
```


**Accessing the API without '*authorization*' Header**

Access rest api at URL: *HTTP GET http://localhost:8080/employees/*
![Require username and password](https://howtodoinjava.com/wp-content/uploads/2018/10/Require-username-and-password.png)

Require username and password


**With `authorization` Header**

Upon passing authorization request header with encoded basic-auth user name and password combination, we will be able to access the rest api response.
Access rest api at URL: *HTTP GET http://localhost:8080/employees/*

![Successful api call](https://howtodoinjava.com/wp-content/uploads/2018/10/Successful-api-call.png)

Successful api call


### Generate Basic Auth Encoding
Browser apI testing tools are able to generate the base-64 encoded token by themselves using the plain username and password. But if we need to generate the encoded token ourselves to pass the token programmatically, then we can use the following code that uses the [java.util.Base64]


**1. Encoding a String to Base64**

This is as simple as getting an instance of the Base64.Encoder and input the string as bytes to encode it.

```
Base64.Encoder encoder = Base64.getEncoder();
String normalString = "username:password";

String encodedString = encoder.encodeToString(
        normalString.getBytes(StandardCharsets.UTF_8) );
```

Output

`dXNlcm5hbWU6cGFzc3dvcmQ=`


**2. Decoding a Base64 Encoded String**

This is also very simple. Just get the instance of Base64.Decoder and use it to decode the base 64 encoded string.

```
String encodedString = "dXNlcm5hbWU6cGFzc3dvcmQ=";
Base64.Decoder decoder = Base64.getDecoder();

byte[] decodedByteArray = decoder.decode(encodedString);

//Verify the decoded string
System.out.println(new String(decodedByteArray));
```

Output

`username:password`


 <br>

# Basic Authentication - with Database & UserDetailsService

A default implementation of the *AuthenticationProvider* uses the default implementations provided for the *UserDetailsService* and the *PasswordEncoder*.

The default implementation of *UserDetailsService* only registers the default credentials in the memory of the application. These default credentials are `"user"` with a default password that's a randomly generated universally unique identifier (UUID) written to the application console when the spring context loads.

```
Using generated security password: 78nh23h-sd56-4b98-86ef-dfas8f8asf8
```

Note that *UserDetailsService* is always associated with a `PasswordEncoder` that encodes a supplied password and verifies if the password matches an existing encoding. When we replace the default implementation of the *UserDetailsService*, we must also specify a *PasswordEncoder*.

In above example we have used In-memory UserDetailsService

```
  @Bean
  public InMemoryUserDetailsManager userDetailsService() {
    UserDetails user = User
        .withUsername("user")
        .password(passwordEncoder().encode("password"))
        .roles("USER_ROLE")
        .build();
    return new InMemoryUserDetailsManager(user);
  }
 ```
