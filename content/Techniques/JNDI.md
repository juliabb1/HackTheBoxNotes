---
tags: [technique, concept, java]
aliases: [Java Naming and Directory Interface]
---
# JNDI (Java Naming and Directory Interface)

> A Java API for looking up objects and resources by name. It is the unified interface in front of several directory services → and it can resolve objects **over the network**, which is what makes it dangerous in [[Log4Shell]].

## What it is
- Java API → "give me the object/resource registered under this name"
```java
DataSource db = (DataSource) ctx.lookup("java:comp/env/jdbc/myDB");
```
- JNDI is only the interface; behind it sit different **directory services**:
  - **LDAP** — directory service (users/computers in companies, Active Directory)
  - **RMI** — Remote Method Invocation (Java objects over the network)
  - **DNS**, and others

## Why it becomes RCE
JNDI can fetch objects not just locally but **over the network**.

**LDAP remote object loading** — an LDAP response can return a *reference* instead of data; older/default-configured clients follow it:
1. LDAP response says: "the class is at `http://server/Evil.class`"
2. Java downloads the class
3. Java instantiates it → the code runs

→ remote code execution, with the attacker controlling both the LDAP server and the HTTP server ([[rogue-jndi]] is exactly this pair).

Used in: [[Log4Shell]]
