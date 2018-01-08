# PipeScript&reg;

This is a Draft Specification for PipeScript&reg;, a DSL (domain specific language) for describing how to orchestrate the flow of data between systems.

## Introduction

PipeScript is captured in the HOCON format and is composed of the following sections:

1. Tasks - defines operations to be performed using the incoming DOM
2. Pipelines - defines flow control of DOMs between Tasks
3. Services - allows for pipeline operations to be exposed as RESTful endpoints
4. Startup - defines which pipeline to execute by default

## Config
The following BNF form of PipeScript&reg; is captured below:

```BNF
<script> ::= 'script {' <sections> '}'

<sections> ::= <task_section> [<pipelines_section>] [<services_section>] <startup>

<task_section> ::= 'tasks { ' <tasks> ' }'
<tasks> ::= <task> ['\n' <tasks>]
<task> ::= <name> ' { ' <task_type> ' }'
<task_type> ::= <task_extract> | <task_transformTerm> | <task_load>

<pipelines_section> ::= 'pipelines { ' <pipelines> ' }'
<pipelines> ::= <pipeline> ['\n' <pipelines>]
<pipeline> ::= <name> ' { pipe = "' <pipeline_expression> '" }'
<pipeline_expression> ::= 

<services_section> ::= 'services = [ ' <services> ' ]'
<services> ::= <service> [',\n' <services>]
<service> ::= '{ path =  "' <url> '" \n' <service_methods> ' }'
<service_methods> ::= <service_method> ['\n' <service_methods>]
<service_method> ::= ('get' | 'put' | 'post' | 'patch' | 'delete') ' = ' <name>


<startup> ::= 'startup { exec = ' <name> ' }'

<name> ::= 
<url> ::= 
<task_extract> ::= 
<task_transformTerm> ::=
<task_load> ::=

```



## Tasks

Tasks can be identified by name and live under the tasks section of the configuration. Each task should have a task type defined which indicates what function it has. Generally, if a task supports different behaviors it will have a behavior defined. For Extractors and Loaders, tasks will require a Data Source to be defined.

Tasks are generally specified as follows:

```HOCON
<task name> {
  type = <task type>
  [behavior = <task behavior>]

  [dataSource {
    ...
  }]
}
```

Each task should appear in the tasks section:

```HOCON
script {
  tasks {
    t1 {
      ...
    }
    t2 {
      ...
    }
    ...
  }
}

```

An example task that extracts data from a RESTful DataSource is shown here:

```HOCON
read_user_api {
  type = extract

  dataSource = ${my_datasource}
  dataSource {
    query {
      read {
        uri = ${my_datasource.base_uri}"/v1/users"
      }
    }
  }
}
```

