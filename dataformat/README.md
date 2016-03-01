# Data format

The data format is somewhat generic and you can use any attributes you need for your use case.
However, we tuned the server backed elasticsearch a bit to support the structure we think is useful.

The examples are all in yaml format. This is just a convience since we think it is easier to read and write than json. A suiteable client  on the other hand must to speak json to the server, so he needs to convert it.

## Where 

You two choices to specify your metadata of your code and services.

- In one single ``pivio.yaml`` file of the root of the source code
- In a directory called ``pivio`` in the root of the source code

The content is structured in both cases the same. In the single file case you need to have section as keys like ``general``, ``network`` and ``software_dependencies``. In the case of a ``pivio`` directory these keys are the file names of the more specialized yaml files. So for each section in the one file you would have corresponding files in the multiple file case. 

So what's the different use cases for both styles? In your own source code you will usually use the directory based approach, since the config file can be very long, especially when you leave all the (useful) comments in. So we split it into much smaller section, so it all should fit on one page in your editor. 
So why use the single file approach at all? For modelling certain aspect of the services you need, like 3rd party services you access, it is very tideous to split the little information you have/need into multiple files. This is usually the case for a single file configuration.

So a config directory version would look like this:

```
+- ...
+- src/
+- pivio/
|   +- network.yaml
|   +- general.yaml
|   +- context.yaml
|   +- runtime.yaml
+- readme.md
+- ...
```
and a single file version more like this:

```
+- ...
+- src/
+- pivio.yaml
+- readme.md
+- ...
```


## What

Every section, except some attributed of the general section, is optional. The idea at the moment is that every section will be represented correspondingly in the UI somehow.

### general.yaml

#### id  (mandantory)
Unique id in pivio. 

#### name (mandantory)
The name of the artefact. This is intended for humans.

#### type  (mandantory)
The type of this artefact. Values could be `service`, `library` or `mobile_app`.

#### owner  (mandantory)
Which team is responsible for this artefact.

#### description  (mandantory)
What does this service do? 

#### contact
Who should be contacted if one has a question.

#### vcs
Where can I find the source code? A client who parses this file might choose to generate it from the code which it has at hand.

#### links
All sort of links which might be interesting. Candidates are

- homepage
- buildchain
- api docs

Example:

```
id: next-generation-print-2342-2413-9189-1990
name: Next Generation Print Service
type: service
owner: Team Goldfinger
description: Prints all kinds of things. Now with 3D printing support.
vcs: git://git.vcs.local/UBP
contact: Auric Goldfinger
links:
  homepage: http://wiki.local/ubp
  buildchain: http://ci.local/ubp
  api_docs: http://docs.local/ubp-api

```

### service.yaml


#### provides
What and where does this artefact provides services? 

`description` Should be a human readable description.
`service_name` is the identification of the particluar interface. `port`, `protocol` and `transportation_protocol` are self describing.

**Question:** Explicit declare host,port,protocol or all together?

#### talks_to
To which other `service_name` (from `provides`) services does this service talk?


Example:

```
provides:
  - description: REST API
    service_name: uber-bill-print-service
    url: https://host:8443
    transportation_protocol: tcp
  - description: SOAP API (legacy)
    service_name: print-service
    url: http://host:80
    transportation_protocol: tcp  
    
talks_to:
  - print-service
  - gateway-service
```

### network.yaml

To which external `target` needs this artefact to talk to? What is the `transportation_protocol` and `why` is it needed? If it access the external resource `via` another service, it can be defined.

Which `attached_networks` need to be available on the host where this service gets deployed. In which `network_zone` is this service located?

```
 external_connections:
   - target: https://api.superdealz.me:443
     transportation_protocol: tcp
     via: proxy-service
     why: Need to sync data with it.
   - target: mqtt://192.xxx.xxx.xxx:5028
     transportation_protocol: tcp
     why: Get the latest Dealz.
     
 attached_networks:
   - logging
   - shell
   - ubp
   - secure
   - monitoring     

network_zone:
   BACKEND
```

### context.yaml

If the service does belong to a bounded context it can be specified in: `belongs_to_bounded_context`. General rule is that every service belongs to a bounded context. 

Which `visibility` does this service have?
 
- `private`: intended usage is only by the owner
- `public`: exposes an api for other owners.

Components that are under development, experimental, not supported, beeing replaces or to change without warning should be `private`.

```
belongs_to_bounded_context: Delivery
visibility: private
```

### runtime.yaml

Which requirements does this service have on the runtime? This is used for sizing the machine, VM or container.

```
runtime:
  cpu: L
  ram: S
  disk: XL
```