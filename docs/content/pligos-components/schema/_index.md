---
title: "Schema Compiler"
date: 2019-10-02T16:09:11+02:00
weight: 1
---

Pligos heavily relies on yaml configurations. In order to compile one set of configurations into different contexts (for example CI,dev,prod) pligos comes with it's own simple schema language to describe the context. The idea is to create a single helm starter that supports a big set of your services. Services then differ only in their templating input (given in `pligos.yaml` file).

Pligos supports the following basic types: `string, numeric, bool, object.` Example:

```
# schema.yaml
context:
  pullPolicy: string
  useTLS: bool
  applicationConfig: object
  podInstances: numeric
```

`contexts`, `flavor` and `spec` are reserved words in following pligos.yaml. You need to provide your schema configuration under `spec`.

```
# example/pligos.yaml -- this is your actual service configuration, define this for all your services individually
contexts:
  dev:
    # Path to flavor folder that contains schema.yaml and templates folder
    flavor: ./flavor
    spec:
      pullPolicy: Always
      useTLS: true
      applicationConfig:
        fixture:
          user: "John Doe"
          balance: "10$"
      podInstances: 1
```

```
# example/values.yaml -- this file will be generated by pligos
pullPolicy: "Always"
useTLS: true
applicationConfig:
  fixture:
    user: "John Doe"
    balance: "10$"
podInstances: 1
```

As mentioned above pligos allows defining different contexts (dev, prod, ...). Each context needs to be set under the `contexts` property in the service configuration.

```
# schema.yaml -- the schema definition for the dev context
context:
  environment: string
  srcPath: string
```

```
# example/pligos.yaml
contexts:
  dev:
    flavor: ./flavor
    spec:
      environment: dev
      srcPath: /home/johndoe/dev/src/
  prod:
    flavor: ./flavor
    spec:
      environment: prod
      srcPath: /home/johndoe/prod/src/
```

Then running the pligos for `prod` will generate the following `values.yaml`

```
# example/values.yaml -- this file will be generated by pligos
environment: prod
srcPath: /home/johndoe/prod/src/
```


However the true power of pligos comes through custom types, which can be instantiated and composed in the configuration.

```
# schema.yaml
route:
  port: string

container:
  route: route

context:
  container: container
```

```
# example/pligos.yaml

contexts:
  dev:
    flavor: ./flavor
    spec:
      container: gowebservice
    
values:
  route:
   - name: http
     port: 80

  container:
   - name: gowebservice
     route: http
```

```
# helmcharts/example/values.yaml
container:
  route:
    port: 80
```

Notice that in the configuration, the instances are referenced by their name. `name` is a special property in pligos which is used for referencering configuration instances (such as the gowebservice container) and `mapped types`. More to `mapped types` later.

Additionally the language supports the meta types `repeated`, `mapped`, `embedded` and `embedded mapped` which can be applied to any custom, or basic types. Let's start with an example for `repeated` instances.

#### Use of `repeated` instance

```
# schema.yaml
container:
  name: string
  command: repeated string

context:
  container: repeated container
```

```
# example/pligos.yaml
contexts:
  dev:
    flavor: ./flavor
    spec:
      container: ["nginx", "php"]

values:
  container:
   - name: nginx
     command: ["nginx"]
   - name: php
     command: ["php-fpm"]

```

```
# examples/values.yaml
container:
 - name: nginx
   command:
    - nginx
 - name: php
   command:
    - php-fpm
```

Repeated allows specifying a list of any type. In the example we have a list of container, as well as a property `command` which is a list of `string`. 

#### Use of `mapped` instance

Next, Pligos also allows you to define maps.

```
# schema.yaml
route:
  name: string
  port: string
  containerPort: string

context:
  routes: mapped route
```

```
# example/pligos.yaml
contexts:
  dev:
    flavor: ./flavor
    spec:
      routes: ["http"]

values:
  route:
   - name: http
     port: 80
     containerPort: 8080

```

```
# example/values.yaml
routes:
  http:
    port: 80
    containerPort: 8080
```

Notice that, although the configuration defines an array of routes, pligos yields a map, as shown in the the output. Maps can be created using the `mapped` meta type.

#### Use of `embedded` instance

Up until this point all custom types appeared under some key in the output. However, this is not always the desired behavior. In order to embed the types' properties into the parent `embedded` types can be used.

```
# schema.yaml
rawValues:
  values: embedded object

context:
  rawValues: embedded rawValues
```

```
# example/pligos.yaml
contexts:
  dev:
    flavor: ./flavor
    spec:
      rawValues: devenvironment

values:
  rawValues:
   - name: devenvironment
     values:
       mysql:
         user: testuser
         password: asdf
```

```
# example/values.yaml
mysql:
  user: testuser
  password: asdf
```

#### Use of `embedded mapped` instance

Similarly the instance can be embedded using any arbitrary key using `embedded mapped` types.

```
# schema.yaml
dependency:
  port: string
  hostname: string

context:
  dependencies: embedded mapped dependency
```

```
# example/pligos.yaml
contexts:
  dev:
    flavor: ./flavor
    spec:
      dependencies: ["mysql"]
    
values:
  dependency:
   - name: mysql
     port: 3306
     hostname: mysql
```

```
# example/values.yaml
mysql:
  port: 3306
  hostname: mysql
```
Just as with `mapped` types, the name property is used to embedd the type instance into the parent.