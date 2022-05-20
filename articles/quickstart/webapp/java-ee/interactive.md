---
title: Add login to your Java EE web application
description: This tutorial demonstrates how to add user login to a Java EE web application.
budicon: 448
topics:
  - quickstarts
  - webapp
  - login
  - java ee
contentType: tutorial
useCase: quickstart
github:
  path: 01-Login
interactive: true
files:
  - files/auth-config
  - files/auth-mechanism
  - files/auth-provider
  - files/jwt-credential
  - files/jwt-identity-store
  - files/jwt-principal
  - files/web
---

# Add login to your Java EE web application

Auth0 allows you to add authentication and gain access to user profile information in different kind of applications. This guide demonstrates how to integrate Auth0 with any new or existing Java EE web application.
 
<%= include('../../_includes/_configure_auth0_interactive', {
callback: 'http://localhost:3000/callback',
returnTo: 'http://localhost:3000/'
}) %>

## Install dependencies {{{ data-action=code data-code="pom.xml" }}}

To integrate your Java EE application with Auth0, add the following dependencies:

- **javax.javaee-api**: The Java EE 8 API necessary to write applications using Java EE 8. The actual implementation is provided by the application container, so it does not need to be included in the WAR file.
- **javax.security.enterprise**: The Java EE 8 Security API that enables handling security concerns in an EE application. Like the `javax.javaee-api` dependency, the implementation is provided by the application container, so is not included in the WAR file.
- **auth0-java-mvc-commons**: The [Auth0 Java MVC SDK](https://github.com/auth0/auth0-java-mvc-common) allows you to use Auth0 with Java for server-side MVC web applications. It generates the Authorize URL that your application needs to call in order to authenticate a user using Auth0.

If you are using Maven, add these dependencies to your `pom.xml` file.

If you are using Gradle, add them to your `build.gradle`:

```java
// build.gradle

providedCompile 'javax:javaee-api:8.0.1'
providedCompile 'javax.security.enterprise:javax.security.enterprise-api:1.0'
implementation 'com.auth0:mvc-auth-commons:1.+'
```

## Configure your application {{{ data-action=code data-code="web.xml" }}}

::: note
The sample that accompanies this tutorial is written using JSP and tested with the [WildFly](https://wildfly.org/) application server. You may need to adjust some of the steps if you working with a different application container or technologies.
:::

Your Java EE application needs some information in order to authenticate users with your Auth0 application. The deployment descriptor `src/main/webapp/WEB-INF/web.xml` file can be used to store this information, though you could store them in a different secured location.

This information will be used to configure the **auth0-java-mvc-commons** library to enable users to login to your application. To learn more about the library, including its various configuration options, see the [README](https://github.com/auth0/auth0-java-mvc-common/blob/master/README.md) of the library.

::: panel Check populated attributes
If you downloaded this sample using the **Download Sample** button, the `domain`, `clientId` and `clientSecret` attributes will be populated for you. You should verify that the values are correct, especially if you have multiple Auth0 applications in your account.
:::

## Configure Java EE Security {{{ data-action=code data-code="Auth0AuthenticationConfig.java" }}}

The Java EE 8 Security API introduced the `HttpAuthenticationMechanism` interface to enable applications to obtain a user's credentials. Default implementations exist for Basic and form-based authentication, and it provides an easy way to configure a custom authentication strategy.

To authenticate with Auth0, provide custom implementations of the following interfaces:

- `HttpAuthenticationMechanism`: Responsible for obtaining a user's credentials and notifying the container of successful (or not) login status ([JavaDoc](https://javaee.github.io/javaee-spec/javadocs/javax/security/enterprise/authentication/mechanism/http/HttpAuthenticationMechanism.html)).
- `IdentityStore`: Responsible for validating the user's credentials ([JavaDoc](https://javaee.github.io/javaee-spec/javadocs/javax/security/enterprise/identitystore/IdentityStore.html)).
- `CallerPrincipal`: Represents the caller principal of the current HTTP request ([JavaDoc](https://javaee.github.io/javaee-spec/javadocs/javax/security/enterprise/CallerPrincipal.html)).
- `Credential`: Represents the credential the caller will use to authenticate ([JavaDoc](https://javaee.github.io/javaee-spec/javadocs/javax/security/enterprise/credential/Credential.html)).


