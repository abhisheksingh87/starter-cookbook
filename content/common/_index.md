+++
categories = ["recipes"]
tags = ["start"]
summary = "Let's get started creating a new microservice"
title = "Let's get started"
date = 2020-12-09
weight = 10
draft = false
authors = ["Prashanth 'PB' Belathur"]
pre ="<i class='fa fa-spinner fa-pulse fa-1x fa-fw'></i>&nbsp;&nbsp;"
+++

Cookbook with recipes for using the greenfield application starters to develop modern cloud native microservice in Wells Fargo.  
The cookbook and starters were created by VMWare Tanzu (Pivotal) Labs during the WellsFargo Enterprise Architecture Consulting engagement in January 2021.


## CONTEXT

The following spring boot starters in Github will help Wells Fargo developers to build _non-reactive_ or _reactive_ microservices based on the business need:
- **greenfield-app-starter** for migrating a _legacy application_ to a cloud-ready microservice.
- **greenfield-reactive-app-starter** for _greenfield applications_, to take advantage of the non-blocking behavior which improves application performance and resiliency.

Both the application starters share a common **Application Tech Stack**, comprising of components approved for use within Wells Fargo.

![Application Tech Stack](../../static/images/tech-stack.png)

## SOLUTION

### [Build Non-Reactive microservice](#non-reactive-path)

Complete the recipes in following order:
- **Create barebone microservice using the starter**
- **Configure Actuators**
- **Configure Oracle**
- **Configure MongoDB**

### [Build Reactive microservice](#reactive-path)

Complete the recipes in following order:
- **Create barebone microservice using the starter**
- **Configure Actuators**
- **Configure MongoDB**

## NOTES
- [What is Reactive Programming ?](https://blog.redelastic.com/what-is-reactive-programming-bc9fa7f4a7fc)
- [Essence of Reactive Programming](https://www.scnsoft.com/blog/java-reactive-programming)