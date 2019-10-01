---
title: "Pligos Documentation"
date: 2019-09-30T17:32:22+02:00
description: This article describes how we can use pligos to manage helm charts easily.
weight: 1
---

<h1>Pligos Documentation</h1>

##### Author: Abdul Jabbar
##### Created at: 19 Sep 2019

### Overview

Pligos allows you to navigate multiple services by making kubernetes
infrastructure configuration scalable. Without pligos, the usual
approach is to create a seperate helm chart for each service. While
this definitely can scale for a small amount of services, maintaining
`deployment, service, ingress, ...` templates for more than 5 helm
charts can be **burdensome** and error prone.

We observed that services, in it's core, often don't differ that
much. You can find a set of configurations that need to be
individually defined for each service, such as `images, routes,
mounts, ...`, so why not standardize around these configuration types,
while beeing disconnected from the underlying templates? This is why
pligos let's you define these configuration types (`image, route,
...`) and adds a schema language that allows you to compile those
configs into any form necessary.

So, what you will end up with is a small set of helm starters (in
pligos lingua franca they are called flavors) and a pligos
configuration for each service that map to these flavors.