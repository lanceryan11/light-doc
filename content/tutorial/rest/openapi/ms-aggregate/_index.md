---
title: "Microservices Aggregate Pattern"
date: 2017-11-29T10:06:31-05:00
description: "How to build Restful microservices in aggregate pattern"
categories: []
keywords: []
menu:
  docs:
    parent: "tutorial"
    weight: 50
weight: 50
aliases: []
toc: false
draft: false
---


This is another pattern that is very useful for serving mobile native applications. The mobile
app just send one request to an aggregate API and it will call multiple APIs to gather info
and send back to the consumer. This avoid mobile device to call multiple APIs to get data on a
slow network. 

This tutorial shows you how to build 4 services with one of them the aggregator. And it will
be the foundation for our microserives benchmarks.

```
API AA -> API AB
      -> API AC
      -> API AD
```

API AA calls API AB, API AC and API AD to fulfill its request.


It is a very long tutorial as we want to package as much info as possible. So we have
separated the it into the following steps. 

- [Environment preparation](/tutorial/rest/openapi/ms-aggregate/preparation/)
- [Create specification](/tutorial/rest/openapi/ms-aggregate/specification/)
- [Generate and Build](/tutorial/rest/openapi/ms-aggregate/generation/)
- [API to API Communication](/tutorial/rest/openapi/ms-aggregate/apitoapi/)
- [Performance Test](/tutorial/rest/openapi/ms-aggregate/performance/)
