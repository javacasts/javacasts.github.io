---
layout:     post
title:      "Spring Security"
date:       2018-08-26 16:10:00 +0200
modified:   2018-08-26 16:10:00 +0200
categories: security
keywords:   java, security, spring security
video:      r8U05bJIDQo
abstract:   "What is spring security doing exactly, and how can you adapt it to your needs?"
author:     "Rainer Jung"
---
Spring Security
===============

I used Spring security for web applications, but so far never did set it up on
myself. So it's time to take a look at on how it is actually working, and at
which spots I can change it for my needs.

Generally Spring Security is about Authentication and Authorization. Amongst
those, responsibilities, there are others but Authentication and Authorization
are the most important duties.  
With the Authentication, the identity of the caller is derived. The *who* is
answered by the Authentication: *who has requested this resource?*.  
With the Authorization, the permission of the caller is derived from the
authentication information. The *what is allowed* by the Authorization: *Is the
caller allowed to execute this?*.

The current version of spring security is 5.x, the classes I talk about could
be different in future releases.

What changes when I enable spring security?
-------------------------------------------

I created a simple application with spring security. I wanted to see, what's the
difference if I start the application with or without spring security.  
The difference seems not to be big when you startup the application. There are
just some additional lines in the bootup output. There are some lines giving you
some security password, and all the other lines talk about some filters and a
filter chain:
```
> [localhost-startStop-1] INFO  o.s.b.w.s.DelegatingFilterProxyRegistrationBean - Mapping filter: 'springSecurityFilterChain' to: [/*]
> [main      ] INFO  o.s.b.a.s.s.UserDetailsServiceAutoConfiguration -
>
> Using generated security password: ddfb3482-d123-4cd8-8173-6921baaf95b5
>
> [main      ] INFO  o.s.s.web.DefaultSecurityFilterChain - Creating filter chain: org.springframework.security.web.util.matcher.AnyRequestMatcher@1, [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@11653e3b, org.springframework.security.web.context.SecurityContextPersistenceFilter@a10c1b5, org.springframework.security.web.header.HeaderWriterFilter@25c5e994, org.springframework.security.web.csrf.CsrfFilter@74fef3f7, org.springframework.security.web.authentication.logout.LogoutFilter@63429932, org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@5cbb84b1, org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@522b2631, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@68e62ca4, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@1a411233, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@187eb9a8, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@715fb77, org.springframework.security.web.session.SessionManagementFilter@2189e7a7, org.springframework.security.web.access.ExceptionTranslationFilter@3b152928, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@2b87581]
```

The filter chain
----------------

The idea of the filter chain is pretty easy. The main interface, that this is
about is `javax.servlet.Filter`. Besides if an `init` and `destroy` method, it
contains the method `public void doFilter(ServletRequest request,
ServletResponse response, FilterChain chain) throws IOException,
ServletException;`  
The main idea is easy, you have stacked several filters above each other, call
the first one with the `doFilter`-method, and it will call the next in the
chain. Once the last has finished, it will go back again through the whole
chain.

So here's an example `Filter` implementation that can explain what a filter is
doing:
```java
@Override
public void doFilter(final ServletRequest request, final ServletResponse response,
        final FilterChain chain) throws IOException, ServletException {
    final long before = System.currentTimeMillis();

    chain.doFilter(request, response);

    final long after = System.currentTimeMillis();
    logger.info("Request {} took {} ms.", ((HttpServletRequest) request).getRequestURI(),
        after - before);
}
```

When the `doFilter` method is called, the `Filter` gets the `ServletRequest`,
`ServletResponse` and the `FilterChain`. It could completely respond to the
`ServletRequest` by sending something to the `ServletResponse` and finishing
itself. By default, it will call `chain.dofilter(request, response)` somewhere
and pass on the processing of the request/response, but each `Filter` could
decide that no further processing should happen. It can also completely modify
the request or add headers to the response.

The filter that I created, just takes the time before it passes on to the
further filter chain, and when returning from it, calculated how many
milliseconds have passed and writes this to the logging.

The spring security filter chain
--------------------------------

From the output of the boot sequence, we've already seen the security filter
chain:

 - `org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter`
 - `org.springframework.security.web.context.SecurityContextPersistenceFilter`
 - `org.springframework.security.web.header.HeaderWriterFilter`
 - `org.springframework.security.web.csrf.CsrfFilter`
 - `org.springframework.security.web.authentication.logout.LogoutFilter`
 - `org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter`
 - `org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter`
 - `org.springframework.security.web.authentication.www.BasicAuthenticationFilter`
 - `org.springframework.security.web.savedrequest.RequestCacheAwareFilter`
 - `org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter`
 - `org.springframework.security.web.authentication.AnonymousAuthenticationFilter`
 - `org.springframework.security.web.session.SessionManagementFilter`
 - `org.springframework.security.web.access.ExceptionTranslationFilter`
 - `org.springframework.security.web.access.intercept.FilterSecurityInterceptor`

Spring boot creates this filter chain from the beans that are available, so the
chain could also be different, but let's go through the relevant filters (as you
are a curious person, you'll want to take a look at the others later).

### `SecurityContextPersistenceFilter`

The [`SecurityContextPersistenceFilter`](https://github.com/spring-projects/spring-security/blob/5.0.x/web/src/main/java/org/springframework/security/web/context/SecurityContextPersistenceFilter.java)
first checks if it was already called in the current request. Then retrieves or
creates the current session and tries to get a `SecurityContext` from it. If
successful, it stores this context in the `SecurityContextHolder` and passes on
the chain.  
When the request was done, it tries to store the `SecurityContext` in the
session and clears the `SecurityContextHolder`.

Here are some classes that we did not talk about before. Let's start with the
`SecurityContext`. The [`SecurityContext`](https://github.com/spring-projects/spring-security/blob/5.0.x/core/src/main/java/org/springframework/security/core/context/SecurityContext.java)
simply contains a [`Authentication`](https://github.com/spring-projects/spring-security/blob/5.0.x/core/src/main/java/org/springframework/security/core/Authentication.java),
what means, it contains information of the users authentication. This could also
mean that the user is not authenticated, but this information is stored in the
`SecurityContext`. As lot of places in the code want to get this information,
it's not passed over as a parameter, it is stored in the [`SecurityContextHolder`](https://github.com/spring-projects/spring-security/blob/5.0.x/core/src/main/java/org/springframework/security/core/context/SecurityContextHolder.java).
The `SecurityContextHolder` can be statically called and returns for each thread
an own instance (see [ThreadLocal](https://docs.oracle.com/javase/8/docs/api/java/lang/ThreadLocal.html)
for more information).

To sum up, the `SecurityContextPersistenceFilter` provides the later filters and
the application with the `Authentication` and as well stores changed
`Authentication` for later requests. It also cleans the `SecurityContextHolder`
so that the next request running through this filter cannot access the
`Authentication` of the current request.

This is also the first potential point, where you could change things. Perhaps
you have multiple servers and cannot or don't want to store the current
`Authentication` in the session. So you can change the
`SecurityContextRepository` to any other implementation.  
Beware of removing the `SecurityContextPersistenceFilter` from the filter chain,
as it clears out the current `Authentication`.

### `CsrfFilter`

The `CsrfFilter` is the first filter that does not take care of Authentication
or Authorization (in combination with the Authentication). This filter is a
protection against Cross-Site Request Forgery, read more about this at
[Wikipedia](https://en.wikipedia.org/wiki/Cross-site_request_forgery). If you
have problems sending requests, this could be an issue.

### `LogoutFilter`

The `LogoutFilter` also is not directly about Authentication or Authorization,
but it's about de-Authentication. You can configure a `RequestMatcher` (we'll
have this later) that can decide for some request that the user now should be
logged out. In this case the `LogoutFilter` will call a `LogoutHandler` (that
you can of course adapt to your needs) that will usually remove the current
authentication. It will then forward the request to a `LogoutSuccessHandler`
that usually forwards the client to a configured ULR, we'll also get to the
configuration later. Of course, it's not required to do a redirect. You can use
and `LogoutSuccessHandler` that does completely different things. In the case of
a logout, the `LogoutFilter` is the first filter that does not always follow the
complete chain. If the `RequestMatcher` matches, the chain will not be called
any further.

### `UsernamePasswordAuthenticationFilter`

Now we're at the Authentication of an unauthorized clients. But the
authentication does not need to be done via a username and password. This is
only the default, if you use the default configuration. You can use any own
implementation, but it's a good idea to use a child of
`AbstractAuthenticationProcessingFilter`, again, you can create an own one.  
The `AbstractAuthenticationProcessingFilter` first checks, if the request needs
any authentication at all. If not, it will not run any further and calls the
further filter chain. If the authentication is required, it gathers username and
password from the request and ask the `AuthenticationManager` for the
`Authentication`.

The `AuthenticationManager` is a very important class. If you want to access an
own database or any other resources, this is what you need to implements, so
let's take a look at the interface. It only contains one method:

```java
Authentication authenticate(Authentication authentication)
    throws AuthenticationException;
```

It takes a `Authentication` and returns an `Authentication`, so here are the
getters of the `Authentication` interface.

```java
String getName();
Collection<? extends GrantedAuthority> getAuthorities();
Object getCredentials();
Object getDetails();
Object getPrincipal();
boolean isAuthenticated();
```

Of course, the interface has a lot more documentation, so take a [look at
it](https://github.com/spring-projects/spring-security/blob/5.0.x/core/src/main/java/org/springframework/security/core/Authentication.java).
As you see, there are a lot of `Object`s, so you can nearly put anything into a
`Authentication`-object. As it is filled with the authentication-filter, you
need to take care that the objects work if you choose to change anything.  
The `UsernamePasswordAuthenticationFilter` creates a
`UsernamePasswordAuthenticationToken` containing the `username` and `password`
to pass this as `credentials` to the `AuthenticationManager`. When the
`AuthenticationManager` returns a `Authentication`, it should be `authenticated`
or not.

There's another special `AuthenticationManager` called the
[`ProviderManager`](https://github.com/spring-projects/spring-security/blob/5.0.x/core/src/main/java/org/springframework/security/authentication/ProviderManager.java).
If you want to check multiple `AuthenticationManager`s, you can implement
`AuthenticationProvider`s instead and have the `ProviderManager` decide, which
authentication to choose.

If you want to use any implementation other than the default, this is the flow
you need to take a look at.

Adapt authentication to your needs
----------------------------------

I've now been able to authenticate myself with the credentials given to my by
the default configuration. This is definitely not the way to go with a real live
application. Usually there is either a user database or an external resource
doing the authentication for you. I'll point you to these two ways. For both
I'll show you the easiest way to achieve your goal, so this is a startingpoint.

### Authenticate against a database

The simpliest way to authorize against a database is to implement a
[`UserDetailsService`](https://github.com/spring-projects/spring-security/blob/5.0.x/core/src/main/java/org/springframework/security/core/userdetails/UserDetailsService.java).
Adding [spring-data-jpa](https://spring.io/guides/gs/accessing-data-jpa/) and a
`Repository` to load the `UserDetails` is already enough. Building a
configuration with an available `AuthenticationProvider` already does the
trick.  
In my example, a in-memory database is used, so I also need to add
a user with each start. Now we already can authorize against the database.

```java
@Bean
public DaoAuthenticationProvider authenticationProvider(
    final UserDetailsService userDetailsService) {
  final DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
  authProvider.setUserDetailsService(userDetailsService);
  authProvider.setPasswordEncoder(encoder());
  return authProvider;
}

@Bean
public PasswordEncoder encoder() {
  return new BCryptPasswordEncoder(11);
}
```

Creating users and updating passwords is another topic that now'd need to be
implemented.

### Authorizing against an OAuth2 provider

