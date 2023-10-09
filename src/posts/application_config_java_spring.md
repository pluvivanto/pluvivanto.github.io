---
title: Application Configuration in Java Spring: application.properties vs application.yml
date: git Created
tags: [java, spring, web]
permalink: "posts/{{ title | slug }}/index.html"
---

Since I started using Spring Boot, I’ve really taken a liking to it. But there’s this quirky habit that’s formed. Every time I grab a new Spring project from the Spring Initializer, I end up deleting the default application.properties file and crafting a new application.yml file. Then, I fill it with stuff like flags, paths, and database settings, just out of habit. What’s funny is that I can’t exactly explain why I do it, but it’s become a part of my routine. Lately, I’ve decided to face my curiosity head-on and delve into the Spring documentation to better grasp this practice.

## Why Use the “application.\*” File?

Actually, I might not need it, as the document says:  
> Spring Boot lets you externalize your configuration so that you can work with the same application code in different environments.

Using application.\* file is called *Externalized Configuration*. This is handy when you want different settings for different things. Like, having special setups for production, development, or testing. And it goes beyond that, covering situations like MSA or Docker environments.  
Keeping configuration apart from code is a smart move in most cases, if not all. Especially in big company codebases, you’d rather not hunt down and modify all the `@PropertySource` notes in your `@Configuration` classes every time you release (and believe me, I’ve been there).

##### code 1. I’m sure you don’t want to edit every `@PropertySource` for each environments
```java
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
 public class AppConfig {
     @Autowired
     Environment env;
     @Bean
     public TestBean testBean() {
         TestBean testBean = new TestBean();
         testBean.setName(env.getProperty("testbean.name"));
         return testBean;
     }
}
```

## So, what’s the deal with .properties and .yml?

When it comes to how they work, they’re basically same. The only difference is the syntax and formatting. Now, let’s check out the example from the official document:

##### code 2: application.properties
```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number-less-than-ten=${random.int(10)}
my.number-in-range=${random.int[1024,65536]}
```

##### code 3: application.yml
```yaml
my:
  secret: "${random.value}"
  number: "${random.int}"
  bignumber: "${random.long}"
  uuid: "${random.uuid}"
  number-less-than-ten: "${random.int(10)}"
  number-in-range: "${random.int[1024,65536]}"
```

## The Verdict

Personally, I’ll stick with .yml over .properties.
Wondering why? Well, yml seems really organized, pushing developers toward a specific coding style (and, honestly, I’m a fan of that).

## Reference:

- https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties.core
- https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config
