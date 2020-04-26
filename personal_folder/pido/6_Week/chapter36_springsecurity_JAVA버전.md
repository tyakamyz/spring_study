
[Part7] - chapter 36
=========================

JAVA 스프링 시큐리티 설정 
----------------

### 예제 jex06

1. Web.xml 을 대신하는 WebConfig.java    
 1) AbstractSecurityWebApplicationInitializer 를 상속받는 클래스 생성
- SecurityInitializer.java
    ```java
    package org.zerock.config;

    import org.springframework.security.web.context.AbstractSecurityWebApplicationInitializer;

    // DelegatingFilterProxy를 스프링에 등록한다.
    public class SecurityInitializer extends AbstractSecurityWebApplicationInitializer{

    }
    ```

2. security-context.xml 을 대신하는 SecurityConfig.java 

    ```java
    package org.zerock.config;


    import javax.sql.DataSource;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
    import org.springframework.security.config.annotation.web.builders.HttpSecurity;
    import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
    import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
    import org.springframework.security.core.userdetails.UserDetailsService;
    import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
    import org.springframework.security.crypto.password.PasswordEncoder;
    import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
    import org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl;
    import org.springframework.security.web.authentication.rememberme.PersistentTokenRepository;
    import org.zerock.security.CustomLoginSuccessHandler;
    import org.zerock.security.CustomUserDetailsService;

    import lombok.Setter;
    import lombok.extern.log4j.Log4j;

    @Configuration
    @EnableWebSecurity  
    @Log4j
    public class SecurityConfig extends WebSecurityConfigurerAdapter{

        @Setter(onMethod_ = { @Autowired })
        private DataSource dataSource;
        
        /*
        // 패스워드 인코딩 테스트 
        @Override
        protected void configure(AuthenticationManagerBuilder auth) throws Exception {
            
            log.info("configure .................................... ");
            
            auth.inMemoryAuthentication().withUser("admin").password("{noop}admin").roles("ADMIN");
            auth.inMemoryAuthentication()
                .withUser("member")
                .password("$2a$10$fog5o0zStJ1KAcSEp.OoTOiSIH6fAN/YDOvL4GGXGV7tZe6ditG4y")
                .roles("MEMBER");
            
        }
        */
        
        @Bean
        public UserDetailsService customUserService() {
            return new CustomUserDetailsService();
        }
        
        // 커스텀 UserdetailsService
        @Override
        protected void configure(AuthenticationManagerBuilder auth) throws Exception {

            auth.userDetailsService(customUserService()).passwordEncoder(passwordEncoder());
        }
        
        /*
        // JDBC를 이용한 로그인 테스트 
        @Override
        protected void configure(AuthenticationManagerBuilder auth) throws Exception {
            log.info("configure JDBC ...................... ");
            
            /*String queryUser = "select userid, userpw, enabled from tbl_member where userid =?";
            String queryDetails = "select userid, auth from tbl_member_auth where userid = ?";
            
            auth.jdbcAuthentication()
                .dataSource(dataSource)
                .passwordEncoder(passwordEncoder())
                .usersByUsernameQuery(queryUser)
                .authoritiesByUsernameQuery(queryDetails);
        }
        */
        
        // 로그인 성공 처리
        @Bean
        public AuthenticationSuccessHandler loginSuccessHandler() {
            return new CustomLoginSuccessHandler();
        }

        @Override
        public void configure(HttpSecurity http) throws Exception {
            
            // 권한 관련 설정
            http.authorizeRequests()
                .antMatchers("/sample/all").permitAll()
                .antMatchers("/sample/admin").access("hasRole('ROLE_ADMIN')")
                .antMatchers("/sample/member").access("hasRole('ROLE_MEMBER')");
            
            // 로그인 페이지 관련 설정 
            http.formLogin()
                .loginPage("/customLogin")
                .loginProcessingUrl("/login");
            
            // 로그아웃 처리
            http.logout()
                .logoutUrl("/customLogout")
                .invalidateHttpSession(true)
                .deleteCookies("remember-me", "JSESSION_ID");
            
            // 자동 로그인 설정 
            http.rememberMe()
                .key("zerock")
                .tokenRepository(persistentTokenRepository())
                .tokenValiditySeconds(604800);
        }

        // PasswordEncoder 지정
        @Bean
        public PasswordEncoder passwordEncoder() {
            return new BCryptPasswordEncoder();
        }
        
        @Bean
        public PersistentTokenRepository persistentTokenRepository() {
            JdbcTokenRepositoryImpl repo = new JdbcTokenRepositoryImpl();
            repo.setDataSource(dataSource);
            return repo;
        }
    ```

    1) @EnableWebSecurity : 스프링MVC와 스프링 시큐리티를 결합하는 어노테이션
    2) configure() 메소드 : security-context.xml의 ```<security:http>``` 관련 설정을 대신한다. 


3. WebConfig.java 변경
    ```java
    package org.zerock.config;

    import javax.servlet.MultipartConfigElement;
    import javax.servlet.ServletRegistration;

    import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

    public class WebConfig extends AbstractAnnotationConfigDispatcherServletInitializer {

        /**
        * root-context.xml 을 대신하는 클래스를 지정 
        */
        @Override
        protected Class<?>[] getRootConfigClasses() {
            return new Class[] {RootConfig.class, SecurityConfig.class};
        }
        ...
    ```
* 스프링이 로딩될 때 SecurityConfig 클래스가 같이 로딩되도록 수정한다. 
