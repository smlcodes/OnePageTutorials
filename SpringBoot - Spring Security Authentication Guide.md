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



### SecurityContext VS JSESSIONID

   For the first time the authentication happens, the JSESSIONID is created & stores inside of the SecurityContext as JSessionID as key & all the authentication information as value.

   from next time as the client sends in the JSESSIONID, AuthenticationFilter uses that JSESSIONID checks if he's already authenticated. And it will not prompt for authentication as long as that cookie is available. And when you deleted it, it will ask for the authentication again.

   Once Authenication is success AuthenticationFilter will return JSESSIONID cookie back to Browser/Postman. If now change the password, it will work because it has valid JSESSIONID in the cookie. If we remove JSESSIONID from postman/browser it shows 401-UnAuthorized. We need to provide valid creds again
 


## Going Deep...
Create a Sample SpringBoot Project & add Maven Dependency.
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```


So you go along, add Spring Security to your Spring Boot (or plain [Spring](https://www.marcobehler.com/guides/spring-framework)) project and suddenly...

-   you have auto-generated login-pages.

-   you cannot execute POST requests anymore.

-   your whole application is on lockdown and prompts you to enter a username and password.

The short answer:

At its core, Spring Security is really just a bunch of servlet filters that help you add [authentication] and [authorization] to your web application.

It also integrates well with frameworks like Spring Web MVC (or [Spring Boot](https://www.marcobehler.com/guides/spring-boot)), as well as with standards like OAuth2 or SAML. And it auto-generates login/logout pages and protects against common exploits like CSRF.


### 1. Authentication

First off, if you are running a typical (web) application, you need your users to *authenticate*. That means your application needs to verify if the user is *who* he claims to be, typically done with a username and password check.

### 2. Authorization
Most applications have the concept of permissions (or roles). Imagine: customers who have access to the public-facing frontend of your webshop, and administrators who have access to a separate admin area.


### 3. Servlet Filters

Last but not least, let's have a look at Servlet Filters

Basically any Spring web application is *just* one servlet: Spring's good old [DispatcherServlet](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-servlet), that redirects incoming HTTP requests (e.g. from a browser) to your **@Controllers** or **@RestControllers**.

The thing is: There is no security hardcoded into that DispatcherServlet and you also very likely don't want to fumble around with a raw HTTP Basic Auth header in your @Controllers. Optimally, the authentication and authorization should be done *before* a request hits your @Controllers.

Luckily, there's a way to do exactly this in the Java web world: you can put [filters](https://www.oracle.com/technetwork/java/filters-137243.html) *in front* of servlets, which means you could think about writing a SecurityFilter and configure it in your Tomcat (servlet container/application server) to filter every incoming HTTP request before it hits your servlet.

![image](https://user-images.githubusercontent.com/20472904/233917882-80261ca8-b688-40b1-b207-33dd94ffb8ee.png)

In the real-world, however, you would split this one filter up into *multiple* filters, that you then *chain* together.

For example, an incoming HTTP request would...​

1.  First, go through a LoginMethodFilter...​

2.  Then, go through an AuthenticationFilter...​

3.  Then, go through an AuthorizationFilter...​

4.  Finally, hit your servlet.

This concept is called *FilterChain* and the last method call in your filter above is actually delegating to that very chain:

```
chain.doFilter(request, response);
```


Let's assume you [set up Spring Security] correctly and then boot up your web application. You'll see the following log message:

```
2020-02-25 10:24:27.875  INFO 11116 --- [           main] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: any request, [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@46320c9a, org.springframework.security.web.context.SecurityContextPersistenceFilter@4d98e41b, org.springframework.security.web.header.HeaderWriterFilter@52bd9a27, org.springframework.security.web.csrf.CsrfFilter@51c65a43, org.springframework.security.web.authentication.logout.LogoutFilter@124d26ba, org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@61e86192, org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@10980560, org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter@32256e68, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@52d0f583, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@5696c927, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@5f025000, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@5e7abaf7, org.springframework.security.web.session.SessionManagementFilter@681c0ae6, org.springframework.security.web.access.ExceptionTranslationFilter@15639d09, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@4f7be6c8]|
```

If you expand that one line into a list, it looks like Spring Security does not just install *one* filter, instead it installs a whole filter chain consisting of 15 (!) different filters.

So, when an HTTPRequest comes in, it will go through *all* these 15 filters, before your request finally hits your @RestControllers. The order is important, too, starting at the top of that list and going down to the bottom.
![image](https://user-images.githubusercontent.com/20472904/233918357-ce61ee28-e80b-48a0-8488-66feec40e9d5.png)

So with these couple of filters, Spring Security provides you a login/logout page, as well as the ability to login with Basic Auth or Form Logins, as well as a couple of additional goodies like the CsrfFilter.Those filters, for a large part, are Spring Security. Hence, we need to have a look at how to configure Spring Security, next.


### How to configure Spring Security: WebSecurityConfigurerAdapter (v2.7.0 VS latest)
In versions prior to Spring Boot 2.7.0, we needed to write a configuration class to inherit WebSecurityConfigurerAdapter, and then rewrite the three methods in the Adapter for configuration.

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

The new usage is very simple, no need to inherit WebSecurityConfigurerAdapter, just declare the configuration class directly. Do below changes

1. First, we'll define our configuration class:

```
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class SecurityConfig {

    // config

}
```


2.we can create a **SecurityFilterChain** bean, in place of configure(Http ) method

```
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.csrf()
      .disable()
      .authorizeRequests()
      .antMatchers(HttpMethod.DELETE)
      .hasRole("ADMIN")
      .antMatchers("/admin/**")
      .hasAnyRole("ADMIN")
      .antMatchers("/user/**")
      .hasAnyRole("USER", "ADMIN")
      .antMatchers("/login/**")
      .anonymous()
      .anyRequest()
      .authenticated()
      .and()
      .httpBasic()
      .and()
      .sessionManagement()
      .sessionCreationPolicy(SessionCreationPolicy.STATELESS);

    return http.build();
}
```

3. **Configure Authentication**: With the *WebSecurityConfigureAdapter, *we'll use an *AuthenticationManagerBuilder* to set our authentication context.
Now, if we want to avoid deprecation, we can define a *UserDetailsManager* or *UserDetailsService *component:

```
@Bean
public UserDetailsService userDetailsService(BCryptPasswordEncoder bCryptPasswordEncoder) {
    InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
    manager.createUser(User.withUsername("user")
      .password(bCryptPasswordEncoder.encode("userPass"))
      .roles("USER")
      .build());
    manager.createUser(User.withUsername("admin")
      .password(bCryptPasswordEncoder.encode("adminPass"))
      .roles("USER", "ADMIN")
      .build());
    return manager;
}
```

Or, given our *UserDetailService*, we can even set an *AuthenticationManager*:


``` 
@Bean
public AuthenticationManager authenticationManager(HttpSecurity http, BCryptPasswordEncoder bCryptPasswordEncoder, UserDetailService userDetailService)
  throws Exception {
    return http.getSharedObject(AuthenticationManagerBuilder.class)
      .userDetailsService(userDetailsService)
      .passwordEncoder(bCryptPasswordEncoder)
      .and()
      .build();
}

```
Similarly, this will work if we use JDBC or LDAP authentication.

# Authentication Types






# 1.Basic Authentication

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

# Ref.
I have gone through below articles while writing this post. So this post not completly wrote by me. 
https://www.marcobehler.com/guides/spring-security#_authentication_with_spring_security


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

# 2.Basic Authentication - with Database & UserDetailsService

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

### Database-backed *UserDetailsService*
To store and retrieve the username and passwords from a SQL database, we use `JdbcUserDetailsManager` class. It connects to the database directly through [JDBC]

By default, it creates two tables in the database:
-   *USERS*
-   *AUTHORITIES*
 

Note that the *JdbcUserDetailsManager* needs a *[DataSource](https://howtodoinjava.com/spring-boot2/datasource-configuration/)* to connect to the database so we need to define it as well.

```
@EnableWebSecurity
public class AppSecurityConfig extends WebSecurityConfigurerAdapter {

  @Bean
  public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
      .setType(EmbeddedDatabaseType.H2)
      .addScript(JdbcDaoImpl.DEFAULT_USER_SCHEMA_DDL_LOCATION)
      .build();
  }

  @Bean
  public UserDetailsService jdbcUserDetailsService(DataSource dataSource) {

    UserDetails user = User
      .withUsername("user")
      .password("password")
      .roles("USER_ROLE")
      .build();

    JdbcUserDetailsManager users = new JdbcUserDetailsManager(dataSource);
    users.createUser(user);
    return users;
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
  }
}
```

We can also write custom SQL queries to fetch the user and authorities' details if we are using a custom DDL schema that uses different table or column names.



```
@Bean
public UserDetailsService jdbcUserDetailsService(DataSource dataSource) {
  String usersByUsernameQuery = "select username, password, enabled from tbl_users where username = ?";
  String authsByUserQuery = "select username, authority from tbl_authorities where username = ?";

  JdbcUserDetailsManager users = new JdbcUserDetailsManager(dataSource);

  userDetailsManager.setUsersByUsernameQuery(usersByUsernameQuery);
  userDetailsManager.setAuthoritiesByUsernameQuery(authsByUserQuery);

  return users;
}
```
 
