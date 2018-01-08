# PipeScript&reg;

This is a Draft Specification for PipeScript&reg;, a DSL (domain specific language) for describing how to orchestrate the flow of data between systems.

##Introduction
PipeScript is captured in the HOCON format and is composed of the following sections:

* Tasks
* Pipelines
* Services
* Startup

These sections will need to be specified under the *script* label:

```HOCON
script {
  tasks {
    ...
  }
  
  pipelines {
    ...
  }
   
  services {
    ...
  }

  startup {
    ...
  }
}

```

