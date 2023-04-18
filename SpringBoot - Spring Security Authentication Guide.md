Spring Boot supports several authentication types to secure your application. Here are the most commonly used authentication types:

1.  **Basic Authentication**: Basic authentication is a simple authentication scheme built into the HTTP protocol. With this type of authentication, the user's credentials are sent in the header of each request.

2.  **OAuth2**: OAuth2 is an open standard for authorization that allows users to grant third-party applications access to their resources without giving them their credentials. Spring Boot provides OAuth2 support using the Spring Security OAuth2 module.

3. **JWT Authentication** (JSON Web Tokens ): JSON Web Tokens are a secure way to transmit information between parties as a JSON object. JWTs are commonly used for authentication and authorization purposes in modern web applications.

4.  **LDAP Authentication**: LDAP authentication allows Spring Boot to authenticate users against an LDAP directory service. This is useful for enterprise applications that already have an LDAP infrastructure in place.

5.  **SAML Authentication**: SAML (Security Assertion Markup Language) is an XML-based standard for exchanging authentication and authorization data between parties. Spring Boot provides support for SAML-based authentication using the Spring Security SAML module.

These are just some of the authentication types supported by Spring Boot. Depending on the requirements of your application, you can choose the appropriate authentication type or even combine multiple types to provide a more secure and flexible authentication mechanism.


# Basic Authentication

Basic Authentication is a simple authentication scheme that sends the user's credentials in the header of each request.  

Basic authentification is a standard HTTP header with the user and password encoded in base64 : `Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==`.The userName and password is encoded in the format `username:password`. This is one of the simplest technique to protect the REST resources because it does not require cookies. session identifiers or any login pages.

1.Add the Spring Security dependency to your project by adding the following code to your `pom.xml` file:  
   
   ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
   ```
   

2.Create a new class and annotate it with `@Configuration` and `@EnableWebSecurity`. This will allow you to configure Spring Security for your application.


3.Extend the `WebSecurityConfigurerAdapter` class and override the `configure(HttpSecurity http)` method to configure your security settings.    
Here we want our every request to be authenticated using HTTP Basic authentication. If authentication fails, the configured **AuthenticationEntryPoint** will be used to retry the authentication process.

 ```java
 
@Configuration
@EnableWebSecurity
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter{

    @Autowired
    private AuthenticationEntryPoint authEntryPoint;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable().authorizeRequests()
                .anyRequest().authenticated()
                .and().httpBasic()
                .authenticationEntryPoint(authEntryPoint);
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().withUser("admin").password("admin").roles("USER");
    }
}
  ```
  
  
  4.Now let us define our authentication entry point.This class will be responsible to send response when the credentials are no longer authorized. If the authentication event was successful, or authentication was not attempted because the HTTP header did not contain a supported authentication request, the filter chain will continue as normal. The only time the filter chain will be interrupted is if authentication fails and the AuthenticationEntryPoint is called.
  ```java
  public class AuthenticationEntryPoint extends BasicAuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authEx)
            throws IOException {
        response.addHeader("WWW-Authenticate", "Basic realm=" +getRealmName());
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        PrintWriter writer = response.getWriter();
        writer.println("HTTP Status 401 - " + authEx.getMessage());
    }

    @Override
    public void afterPropertiesSet()  {
        setRealmName("DeveloperStack");
        super.afterPropertiesSet();
    }

}
  ```
  
 5. After a succesdfull authentication, Spring updates the security context with an authentication object that contains credentials, roles, principal etc.

 6. Check API calls to validate Basic Authentication. If we open in browser it will ask for credencisals. If you use postman
 7. <img width="1095" alt="image" src="https://user-images.githubusercontent.com/20472904/232781385-6c57e140-8c0c-4b12-80c0-b783b9c03720.png">


  
  Following logout implementation is one way to logout programatically.You can also configure logout in SpringSecurityConfig.java.You can use any one of them but not both.Once basic authentication is successfull, browser automatically sends basic auth header in every request following successfull authentication.Hence, if you are using this authentication mechanism in a browser based application, it is required to clear the Basic auth cache in the browser too after logout.
  ```java
  @RequestMapping(value="/logmeout", method = RequestMethod.POST)
public String logoutPage (HttpServletRequest request, HttpServletResponse response) {
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
if (auth != null){
new SecurityContextLogoutHandler().logout(request, response, auth);
}
return "redirect:/login";
}
  
  ```
  
  
  # REF.
  https://www.devglan.com/spring-security/spring-boot-security-rest-basic-authentication
