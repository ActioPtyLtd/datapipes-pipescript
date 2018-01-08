# PipeScript&reg;

This is a Draft Specification for PipeScript&reg;, a DSL (domain specific language) for describing how to orchestrate the flow of data between systems.

## Introduction

PipeScript is captured in the HOCON format and is composed of the following sections:

1. Tasks - defines operations to be performed using the incoming DOM
2. Pipelines - defines flow control of DOMs between Tasks
3. Services - allows for pipeline operations to be exposed as RESTful endpoints
4. Startup - defines which pipeline to execute by default

## Config - Sections
The following BNF form of PipeScript&reg; is captured below:

```BNF
<script> ::= 'script {' <sections> '}'

<sections> ::= <task_section> [<pipelines_section>] [<services_section>] <startup>

```

## Config - Task Section
Tasks can be identified by name and live under the tasks section of the configuration. Each task should have a task type defined which indicates what function it has. Generally, if a task supports different behaviors it will have a behavior defined. For Extractors and Loaders, tasks will require a Data Source to be defined.

The following BNF describes the supported syntax for tasks:
```BNF
<task_section> ::= 'tasks { ' <tasks> ' }'
<tasks> ::= <task> [<tasks>]
<task> ::= <name> ' { type = "' <text> '"' [<key_values>] [<datasource_section>] '}'
```

The following Task types supported are explained below:

### Task Extract
To extract data from a data source, you will need the following definition:
```
type = extract
dataSource {
  ...
  query {
    read {
      ...
    }
  }
}
```

### Task Load
To load data into a data source, you will need the following definition:
```
type = load
dataSource {
  ...
  query {
    create {
      ...
    }
  }
}
```

### Task TransformTerm



## Config - DataSources Section
```BNF
<datasource_section> ::= 'dataSource { ' <datasource> ' }'
<datasource> ::= 'type = "' <text> '"' [<key_values>] [query_section]

<query_section> ::= 'query {' <datasource_queries> '}'
<datasource_queries> ::= <datasource_query> [<datasource_queries>]
<datasource_query> ::= <name> ' {' <templates> '}'

<templates> ::= <name> ' = "' <template> '"' [<templates>] 
<template> ::= <text> [('$'<expression> | '${'<expression>'}')] <template>

```

## Config - Expressions
```BNF
<expression> ::= 
  <literal> | 
  <variable> | 
  <select> | 
  <apply> | 
  <user_function> | 
  <operator_expression> | 
  <if_statement>

<literal> ::= <number> | ('"' <text> '"') | 'false' | 'true'
<variable> ::= <name>
<apply> ::= <name> '()'
<user_function> ::= <name> '(' <args> ')'
<args> ::= <expression> [',' <expression>]
<select> ::= <expression> '.' (<variable> | <apply>)
<operator_expression> ::= <expression> <operator> <expression>
<operator> ::= '+' | '-' | '*' | '/' | '&&' | '||' | '<' | '>' | '<=' | '>=' | '->' | '==' | '!' | '!='
<if_statement> ::= 'if(' <expression> ')' ['elseif(' <expression> ')'] 'else' <expression>
```

## Config - Pipeline Section
```BNF
<pipelines_section> ::= 'pipelines { ' <pipelines> ' }'
<pipelines> ::= <pipeline> [<pipelines>]
<pipeline> ::= <name> ' { pipe = "' <pipeline_expression> '" }'
<pipeline_expression> ::= <name> ['(' <args> ')'] [<pipeline_operator> <pipeline_expression>]
<pipeline_operator> ::= ('|' | '&')
```

## Config - Services Section
```BNF
<services_section> ::= 'services = [ ' <services> ' ]'
<services> ::= <service> [',' <services>]
<service> ::= '{ path =  "' <url> '"' <service_methods> ' }'
<service_methods> ::= <service_method> [<service_methods>]
<service_method> ::= ('get' | 'put' | 'post' | 'patch' | 'delete') ' = ' <name>
```

## Config - Common
```BNF

<startup> ::= 'startup { exec = ' <name> ' }'

<key_values> ::= <name>
<name> ::= ;alphanumeric text
<value> := ;text or number
<args> ::= <name>

<url> ::= ;text with optional variables

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

