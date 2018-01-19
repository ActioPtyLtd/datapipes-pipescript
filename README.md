# PipeScript&reg;

This is a Draft Specification for PipeScript&reg;, a DSL (domain specific language) for describing how to orchestrate the flow of data between systems.

## Introduction

PipeScript is captured in the HOCON format and is composed of the following sections:

1. [Tasks](#task-section) - defines operations to be performed using the incoming DOM
2. [Pipelines](#pipeline-section) - defines flow control of DOMs between Tasks
3. [Services](#services-section) - allows for pipeline operations to be exposed as RESTful endpoints
4. [Startup](#startup-section) - defines which pipeline to execute by default

## Sections
The following BNF form of PipeScript&reg; is captured below:

```BNF
<script> ::= 'script {' <sections> '}'

<sections> ::= <task_section> [<pipelines_section>] [<services_section>] <startup>

```

## Task Section
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
```
type = transformTerm
behavior = ('batch' | 'expand')
term = '"' <expression> '"'
```

### Task MergeLoad
```
type = mergeLoad
entity = <entity_name>
keys = '[' <columns> ']'
update = ('true' | 'false')

dataSource {
  ...
}
```

## DataSources Section
```BNF
<datasource_section> ::= 'dataSource { ' <datasource> ' }'
<datasource> ::= 'type = "' <text> '"' [<key_values>] [query_section]

<query_section> ::= 'query {' <datasource_queries> '}'
<datasource_queries> ::= <datasource_query> [<datasource_queries>]
<datasource_query> ::= <name> ' {' <templates> '}'

<templates> ::= <name> ' = "' <template> '"' [<templates>] 
<template> ::= <text> [('$'<expression> | '${'<expression>'}')] <template>

```

## Expressions
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

### DataSet Functions
The functions that every DataSet have, independant of its type, are listed below:

Converting the DataSet to a String (named 'ds') can be done with the following function:
```javascript
ds.toString()
```

Converting the DataSet to a String in JSON format can be done with the following function:
```javascript
ds.toJson()

```

Converting the DataSet to a String in XML format can be done with the following function:
```javascript
ds.toXml()
```

Checking whether a DataSet has a property value defined, returning a Bool, is as follows:
```javascript
ds.isDefined()
```

Checking whether a DataSet has child elements, returning a Bool, is as follows:
```javascript
ds.isEmpty()
```

## Pipeline Section
```BNF
<pipelines_section> ::= 'pipelines { ' <pipelines> ' }'
<pipelines> ::= <pipeline> [<pipelines>]
<pipeline> ::= <name> ' { pipe = "' <pipeline_expression> '" }'
<pipeline_expression> ::= <name> ['(' <args> ')'] [<pipeline_operator> <pipeline_expression>]
<pipeline_operator> ::= ('|' | '&')
```

## Services Section
The Services Section allows one to define API endpoints and the appropriate routing that needs to occur for different http methods. The BNF form is shown below:

```BNF
<services_section> ::= 'services = [ ' <services> ' ]'
<services> ::= <service> [',' <services>]
<service> ::= '{ path =  "' <url> '"' <service_methods> ' }'
<service_methods> ::= <service_method> [<service_methods>]
<service_method> ::= ('get' | 'put' | 'post' | 'patch' | 'delete') ' = ' <name>
```
Note <name> here is the name of the pipename to start if the spefific method is called.

Example:

```HOCON
services = [
  {
    path = "/api/v1/users"
    get = get_users
  },
  {
    path = "/api/v1/user/$userid"
    get = get_user
    put = update_user
  }
]
```

This configuration will provide access to two endpoints, one to extract users by calling the get_users pipeline and another that allows for a specific user to be retrieved or updated. Note the userid parameter will be extracted and passed onto either pipeline. For the update_user pipeline, the http request as well as the userid parameter will be provided.


## Startup Section
The Startup Section allows one to specify the default pipeline to execute if no specific pipeline is specified on the command line. The BNF is as follows:

```BNF
<startup> ::= 'startup { exec = ' <name> ' }'
```

Note: Here <name> is the default pipeline to execute.

## Common
```BNF

<startup> ::= 'startup { exec = ' <name> ' }'

<key_values> ::= <name>
<name> ::= ;alphanumeric text
<value> := ;text or number
<args> ::= <name>

<url> ::= ;text with optional variables

```


