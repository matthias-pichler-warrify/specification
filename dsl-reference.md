# Serverless Workflow DSL - Reference

## Table of Contents

- [Abstract](#abstract)
- [Definitions](#definitions)
  - [Workflow](#workflow)
    - [Document](#document)
    - [Use](#use)
    - [Schedule](#schedule)
  - [Task](#task)
    - [Call](#call)
      - [AsyncAPI](#asyncapi-call)
      - [gRPC](#grpc-call)
      - [HTTP](#http-call)
      - [OpenAPI](#openapi-call)
    - [Composite](#composite)
    - [Emit](#emit)
    - [For](#for)
    - [Listen](#listen)
    - [Raise](#raise)
    - [Run](#run)
      - [Container](#container-process)
      - [Shell](#shell-process)
      - [Script](#script-process)
      - [Workflow](#workflow-process)
    - [Switch](#switch)
    - [Set](#set)
    - [Try](#try)
    - [Wait](#wait)
  - [Flow Directive](#flow-directive)
  - [External Resource](#external-resource)
  - [Authentication](#authentication)
    - [Basic](#basic-authentication)
    - [Bearer](#bearer-authentication)
    - [Certificate](#certificate-authentication)
    - [Digest](#digest-authentication)
    - [OAUTH2](#oauth2-authentication)
  - [Extension](#extension)
  - [Error](#error)
    - [Standard Error Types](#standard-error-types)
  - [Event Consumption Strategy](#event-consumption-strategy)
  - [Event Filter](#event-filter)
  - [Retry](#retry)
  - [Input](#input)
  - [Output](#output)
  - [Timeout](#timeout)
  - [Duration](#duration)
  - [HTTP Response](#http-response)
  - [HTTP Request](#http-request)

## Abstract

This document provides comprehensive definitions and detailed property tables for all the concepts discussed in the Serverless Workflow DSL. It serves as a reference guide, explaining the structure, components, and configurations available within the DSL. By exploring this document, users will gain a thorough understanding of how to define, configure, and manage workflows, including task definitions, flow directives, and state transitions. This foundational knowledge will enable users to effectively utilize the DSL for orchestrating serverless functions and automating processes.

## Definitions

### Workflow

A [workflow](#workflow) serves as a blueprint outlining the series of [tasks](#task) required to execute a specific business operation. It details the sequence in which [tasks](#task) must be completed, guiding users through the process from start to finish, and helps streamline operations, ensure consistency, and optimize efficiency within an organization.

#### Properties

| Name     |             Type             | Required | Description                                                               |
| :------- | :--------------------------: | :------: | :------------------------------------------------------------------------ |
| document |   [`document`](#document)    |  `yes`   | Documents the defined workflow.                                           |
| input    |      [`input`](#input)       |   `no`   | Configures the workflow's input.                                          |
| use      |        [`use`](#use)         |   `no`   | Defines the workflow's reusable components, if any.                       |
| do       | [`map[string, task]`](#task) |  `yes`   | The [task(s)](#task) that must be performed by the [workflow](#workflow). |
| timeout  |    [`timeout`](#timeout)     |   `no`   | The configuration, if any, of the workflow's timeout.                     |
| output   |     [`output`](#output)      |   `no`   | Configures the workflow's output.                                         |
| schedule |   [`schedule`](#schedule)    |   `no`   | Configures the workflow's schedule, if any.                               |
| evaluate |   [`evaluate`](#evaluate)    |   `no`   | Configures runtime expression evaluation.                                 |

#### Document

Documents the workflow definition.

| Name      |         Type          | Required | Description                                         |
| :-------- | :-------------------: | :------: | :-------------------------------------------------- |
| dsl       |       `string`        |  `yes`   | The version of the DSL used to define the workflow. |
| namespace |       `string`        |  `yes`   | The workflow's namespace.<br>                       |
| name      |       `string`        |  `yes`   | The workflow's name.<br>                            |
| version   |       `string`        |  `yes`   | The workflow's [semantic version]                   |
| title     |       `string`        |   `no`   | The workflow's title.                               |
| summary   |       `string`        |   `no`   | The workflow's Markdown summary.                    |
| tags      | `map[string, string]` |   `no`   | A key/value mapping of the workflow's tags, if any. |

#### Use

Defines the workflow's reusable components.

| Name            |                       Type                       | Required | Description                                                              |
| :-------------- | :----------------------------------------------: | :------: | :----------------------------------------------------------------------- |
| authentications | [`map[string, authentication]`](#authentication) |   `no`   | A name/value mapping of the workflow's reusable authentication policies. |
| errors          |          [`map[string, error]`](#error)          |   `no`   | A name/value mapping of the workflow's reusable errors.                  |
| extensions      |      [`map[string, extension]`](#extension)      |   `no`   | A name/value mapping of the workflow's reusable extensions.              |
| functions       |           [`map[string, task]`](#task)           |   `no`   | A name/value mapping of the workflow's reusable tasks.                   |
| retries         |       [`map[string, retryPolicy]`](#retry)       |   `no`   | A name/value mapping of the workflow's reusable retry policies.          |
| secrets         |                    `string[]`                    |   `no`   | A list containing the workflow's secrets.                                |

#### Schedule

Configures the schedule of a workflow.

| Name  |                           Type                            | Required | Description                                                                                                                                                                                                                                               |
| :---- | :-------------------------------------------------------: | :------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| every |                  [`duration`](#duration)                  |   `no`   | Specifies the duration of the interval at which the workflow should be executed. Unlike `after`, this option will run the workflow regardless of whether the previous run is still in progress.<br>_Required when no other property has been set._        |
| cron  |                         `string`                          |   `no`   | Specifies the schedule using a CRON expression, e.g., '0 0 \* \* *' for daily at midnight.<br>*Required when no other property has been set.\*                                                                                                            |
| after |                  [`duration`](#duration)                  |   `no`   | Specifies a delay duration that the workflow must wait before starting again after it completes. In other words, when this workflow completes, it should run again after the specified amount of time.<br>_Required when no other property has been set._ |
| on    | [`eventConsumptionStrategy`](#event-consumption-strategy) |   `no`   | Specifies the events that trigger the workflow execution.<br>_Required when no other property has been set._                                                                                                                                              |

#### Evaluate

Configures a workflow's runtime expression evaluation.

| Name     |   Type   | Required | Description                                                                                                                                                                                                                                                                                                                                  |
| :------- | :------: | :------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| language | `string` |  `yes`   | The language used for writting runtime expressions.<br>_Defaults to `jq`._                                                                                                                                                                                                                                                                   |
| mode     | `string` |  `yes`   | The runtime expression evaluation mode.<br>_Supported values are:_<br>- `strict`: requires all expressions to be enclosed within `${ }` for proper identification and evaluation.<br>- `loose`: evaluates any value provided. If the evaluation fails, it results in a string with the expression as its content.<br>_Defaults to `strict`._ |

#### Examples

```yaml
document:
  dsl: '1.0.0-alpha1'
  namespace: test
  name: order-pet
  version: '1.0.0'
  title: Order Pet - 1.0.0
  summary: >
  # Order Pet - 1.0.0
  ## Table of Contents
  - [Description](#description)
  - [Requirements](#requirements)
  ## Description
  A sample workflow used to process an hypothetic pet order using the [PetStore API](https://petstore.swagger.io/)
  ## Requirements
  ### Secrets
  - my-oauth2-secret
use:
  authentications:
    petStoreOAuth2:
      oauth2: my-oauth2-secret
  extensions:
    externalLogging:
      extend: all
      before:
        call: http
        with:
          method: post
          endpoint: https://fake.log.collector.com
          body:
            message: "${ \"Executing task '\($task.reference)'...\" }"
      after:
        call: http
        with:
          method: post
          endpoint: https://fake.log.collector.com
          body:
            message: "${ \"Executed task '\($task.reference)'...\" }"
  functions:
    getAvailablePets:
      call: openapi
      with:
        document:
          uri: https://petstore.swagger.io/v2/swagger.json
        operation: findByStatus
        parameters:
          status: available
  secrets:
  - my-oauth2-secret
do:
  getAvailablePets:
    call: getAvailablePets
    output:
      from: "$input + { availablePets: [.[] | select(.category.name == "dog" and (.tags[] | .breed == $input.order.breed))] }"
  submitMatchesByMail:
    call: http
    with:
      method: post
      endpoint:
        uri: https://fake.smtp.service.com/email/send
        authentication: petStoreOAuth2
      body:
        from: noreply@fake.petstore.com
        to: ${ .order.client.email }
        subject: Candidates for Adoption
        body: >
        Hello ${ .order.client.preferredDisplayName }!

        Following your interest to adopt a dog, here is a list of candidates that you might be interested in:

        ${ .pets | map("-\(.name)") | join("\n") }

        Please do not hesistate to contact us at info@fake.petstore.com if your have questions.

        Hope to hear from you soon!

        ----------------------------------------------------------------------------------------------
        DO NOT REPLY
        ----------------------------------------------------------------------------------------------
```

### Task

A task within a [workflow](#workflow) represents a discrete unit of work that contributes to achieving the overall objectives defined by the [workflow](#workflow).

It encapsulates a specific action or set of actions that need to be executed in a predefined order to advance the workflow towards its completion.

[Tasks](#task) are designed to be modular and focused, each serving a distinct purpose within the broader context of the [workflow](#workflow).

By breaking down the [workflow](#workflow) into manageable [tasks](#task), organizations can effectively coordinate and track progress, enabling efficient collaboration and ensuring that work is completed in a structured and organized manner.

The Serverless Workflow DSL defines a list of [tasks](#task) that **must be** supported by all runtimes:

- [Call](#call), used to call services and/or functions.
- [Composite](#composite), used to define a minimum of two subtasks to perform.
- [Emit](#emit), used to emit [events](#event).
- [For](#for), used to iterate over a collection of items, and conditionally perform a task for each of them.
- [Listen](#listen), used to listen for an [event](#event) or more.
- [Raise](#raise), used to raise an [error](#error) and potentially fault the [workflow](#workflow).
- [Run](#run), used to run a [container](#container), a [script](#script) or event a [shell](#shell) command.
- [Switch](#switch), used to dynamically select and execute one of multiple alternative paths based on specified conditions
- [Set](#set), used to dynamically set or update the [workflow](#workflow)'s data during the its execution.
- [Try](#try), used to attempt executing a specified [task](#task), and to handle any resulting [errors](#error) gracefully, allowing the [workflow](#workflow) to continue without interruption.
- [Wait](#wait), used to pause or wait for a specified duration before proceeding to the next task.

#### Properties

| Name    |                Type                | Required | Description                                                                       |
| :------ | :--------------------------------: | :------: | :-------------------------------------------------------------------------------- |
| input   |         [`input`](#input)          |   `no`   | An object used to customize the task's input and to document its schema, if any.  |
| output  |        [`output`](#output)         |   `no`   | An object used to customize the task's output and to document its schema, if any. |
| timeout |       [`timeout`](#timeout)        |   `no`   | The configuration of the task's timeout, if any.                                  |
| then    | [`flowDirective`](#flow-directive) |   `no`   | The flow directive to execute next.<br>_If not set, defaults to `continue`._      |

#### Call

Enables the execution of a specified function within a workflow, allowing seamless integration with custom business logic or external services.

##### Properties

| Name |   Type   | Required | Description                                                      |
| :--- | :------: | :------: | :--------------------------------------------------------------- |
| call | `string` |  `yes`   | The name of the function to call.                                |
| with |  `map`   |   `no`   | A name/value mapping of the parameters to call the function with |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  getPetById:
    call: http
    with:
      method: get
      endpoint: https://petstore.swagger.io/v2/pet/{petId}
```

Serverless Workflow defines several default functions that **MUST** be supported by all implementations and runtimes:

- [AsyncAPI](#asyncapi)
- [gRPC](#grpc-call)
- [HTTP](#http-call)
- [OpenAPI](#openapi-call)

##### AsyncAPI Call

The [AsyncAPI Call](#asyncapi-call) enables workflows to interact with external services described by [AsyncAPI](https://www.asyncapi.com/).

###### Properties

| Name           |                      Type                       | Required | Description                                                                                                                                         |
| :------------- | :---------------------------------------------: | :------: | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| document       |    [`externalResource`](#external-resource)     |  `yes`   | The AsyncAPI document that defines the operation to call.                                                                                           |
| operationRef   |                    `string`                     |  `yes`   | A reference to the AsyncAPI operation to call.                                                                                                      |
| server         |                    `string`                     |   `no`   | A reference to the server to call the specified AsyncAPI operation on.<br>If not set, default to the first server matching the operation's channel. |
| message        |                    `string`                     |   `no`   | The name of the message to use. <br>If not set, defaults to the first message defined by the operation.                                             |
| binding        |                    `string`                     |   `no`   | The name of the binding to use. <br>If not set, defaults to the first binding defined by the operation                                              |
| payload        |                      `any`                      |   `no`   | The operation's payload, as defined by the configured message                                                                                       |
| authentication | `string`<br>[`authentication`](#authentication) |   `no`   | The authentication policy, or the name of the authentication policy, to use when calling the AsyncAPI operation.                                    |

###### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  greetUser:
    call: asyncapi
    with:
      document: https://fake.com/docs/asyncapi.json
      operation: findPetsByStatus
      server: staging
      message: getPetByStatusQuery
      binding: http
      payload:
        petId: ${ .pet.id }
```

##### gRPC Call

The [gRPC Call](#grpc-call) enables communication with external systems via the gRPC protocol, enabling efficient and reliable communication between distributed components.

###### Properties

| Name                   |                   Type                   | Required | Description                                                                                                |
| :--------------------- | :--------------------------------------: | :------: | :--------------------------------------------------------------------------------------------------------- |
| proto                  | [`externalResource`](#external-resource) |  `yes`   | The proto resource that describes the GRPC service to call.                                                |
| service.name           |                 `string`                 |  `yes`   | The name of the GRPC service to call.                                                                      |
| service.host           |                 `string`                 |  `yes`   | The hostname of the GRPC service to call.                                                                  |
| service.port           |                `integer`                 |   `no`   | The port number of the GRPC service to call.                                                               |
| service.authentication |   [`authentication`](#authentication)    |   `no`   | The authentication policy, or the name of the authentication policy, to use when calling the GRPC service. |
| method                 |                 `string`                 |  `yes`   | The name of the GRPC service method to call.                                                               |
| arguments              |                  `map`                   |   `no`   | A name/value mapping of the method call's arguments, if any.                                               |

###### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  greetUser:
    call: grpc
    with:
      proto: file://app/greet.proto
      service:
        name: GreeterApi.Greeter
        host: localhost
        port: 5011
      method: SayHello
      arguments:
        name: ${ .user.preferredDisplayName }
```

##### HTTP Call

The [HTTP Call](#http-call) enables workflows to interact with external services over HTTP.

###### Properties

| Name     |          Type           | Required | Description                                                                                                                                                                                                                                                                                                                                                          |
| :------- | :---------------------: | :------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| method   |        `string`         |  `yes`   | The HTTP request method.                                                                                                                                                                                                                                                                                                                                             |
| endpoint | [`endpoint`](#endpoint) |  `yes`   | An URI or an object that describes the HTTP endpoint to call.                                                                                                                                                                                                                                                                                                        |
| headers  |          `map`          |   `no`   | A name/value mapping of the HTTP headers to use, if any.                                                                                                                                                                                                                                                                                                             |
| body     |          `any`          |   `no`   | The HTTP request body, if any.                                                                                                                                                                                                                                                                                                                                       |
| output   |        `string`         |   `no`   | The http call's output format.<br>_Supported values are:_<br>_- `raw`, which output's the base-64 encoded [http response](#http-response) content, if any._<br>_- `content`, which outputs the content of [http response](#http-response), possibly deserialized._<br>_- `response`, which outputs the [http response](#http-response)._<br>_Defaults to `content`._ |

###### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  getPetById:
    call: http
    with:
      method: get
      endpoint: https://petstore.swagger.io/v2/pet/{petId}
```

##### OpenAPI Call

The [OpenAPI Call](#openapi-call) enables workflows to interact with external services described by [OpenAPI](https://www.openapis.org).

###### Properties

| Name           |                   Type                   | Required | Description                                                                                                                                                                                                                                                                                                                                                             |
| :------------- | :--------------------------------------: | :------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| document       | [`externalResource`](#external-resource) |  `yes`   | The OpenAPI document that defines the operation to call.                                                                                                                                                                                                                                                                                                                |
| operationId    |                 `string`                 |  `yes`   | The id of the OpenAPI operation to call.                                                                                                                                                                                                                                                                                                                                |
| arguments      |                  `map`                   |   `no`   | A name/value mapping of the parameters, if any, of the OpenAPI operation to call.                                                                                                                                                                                                                                                                                       |
| authentication |   [`authentication`](#authentication)    |   `no`   | The authentication policy, or the name of the authentication policy, to use when calling the OpenAPI operation.                                                                                                                                                                                                                                                         |
| output         |                 `string`                 |   `no`   | The OpenAPI call's output format.<br>_Supported values are:_<br>_- `raw`, which output's the base-64 encoded [http response](#http-response) content, if any._<br>_- `content`, which outputs the content of [http response](#http-response), possibly deserialized._<br>_- `response`, which outputs the [http response](#http-response)._<br>_Defaults to `content`._ |

###### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  getPets:
    call: openapi
    with:
      document: https://petstore.swagger.io/v2/swagger.json
      operation: findPetsByStatus
      parameters:
        status: available
```

#### Composite

Serves as a pivotal orchestrator within workflow systems, enabling the seamless integration and execution of multiple subtasks to accomplish complex operations. By encapsulating and coordinating various subtasks, this task type facilitates the efficient execution of intricate workflows.

##### Properties

| Name                 |             Type             | Required | Description                                                                                                                                                                                                                                                                                                          |
| :------------------- | :--------------------------: | :------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| execute.sequentially | [`map(string, task)`](#task) |   `no`   | The tasks to perform sequentially.<br>_Required if `execute.concurrently` has not been set, otherwise ignored._<br>_If set, must contains **at least** two [`tasks`](#task)._                                                                                                                                        |
| execute.concurrently | [`map(string, task)`](#task) |   `no`   | The tasks to perform concurrently.<br>_Required if `execute.sequentially` has not been set, otherwise ignored._<br>_If set, must contains **at least** two [`tasks`](#task)._                                                                                                                                        |
| execute.compete      |          `boolean`           |   `no`   | Indicates whether or not the concurrent [`tasks`](#task) are racing against each other, with a single possible winner, which sets the composite task's output.<br>_Ignored if `execute.sequentially` has been set. Defaults to `false`._<br>_Must **not** be set if the [`tasks`](#task) are executed sequentially._ |

##### Examples

_Executing tasks sequentially:_

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  bookTrip:
    execute:
      sequentially:
        bookHotel:
          call: http
          with:
            method: post
            endpoint:
              uri: https://fake-booking-agency.com/hotels/book
              authentication: fake-booking-agency-oauth2
            body:
              name: Four Seasons
              city: Antwerp
              country: Belgium
        bookFlight:
          call: http
          with:
            method: post
            endpoint:
              uri: https://fake-booking-agency.com/flights/book
              authentication: fake-booking-agency-oauth2
            body:
              departure:
                date: "01/01/26"
                time: "07:25:00"
                from:
                  airport: BRU
                  city: Zaventem
                  country: Belgium
              arrival:
                date: "01/01/26"
                time: "11:12:00"
                to:
                  airport: LIS
                  city: Lisbon
                  country: Portugal
```

_Executing tasks concurrently:_

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  raiseAlarm:
    execute:
      concurrently:
        callNurse:
          call: http
          with:
            method: put
            endpoint: https://fake-hospital.com/api/v3/alert/nurses
            body:
              patientId: ${ .patient.fullName }
              room: ${ .room.number }
        callDoctor:
          call: http
          with:
            method: put
            endpoint: https://fake-hospital.com/api/v3/alert/doctor
            body:
              patientId: ${ .patient.fullName }
              room: ${ .room.number }
```

#### Emit

Allows workflows to publish events to event brokers or messaging systems, facilitating communication and coordination between different components and services. With the Emit task, workflows can seamlessly integrate with event-driven architectures, enabling real-time processing, event-driven decision-making, and reactive behavior based on incoming events.

##### Properties

| Name       |       Type        | Required | Description                |
| :--------- | :---------------: | :------: | :------------------------- |
| emit.event | [`event`](#event) |  `yes`   | Defines the event to emit. |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  placeOrder:
    emit:
      event:
        with:
          source: https://petstore.com
          type: com.petstore.order.placed.v1
          data:
            client:
              firstName: Cruella
              lastName: de Vil
            items:
              - breed: dalmatian
                quantity: 101
```

#### For

Allows workflows to iterate over a collection of items, executing a defined set of subtasks for each item in the collection. This task type is instrumental in handling scenarios such as batch processing, data transformation, and repetitive operations across datasets.

##### Properties

| Name     |      Type       | Required | Description                                                                                                                         |
| :------- | :-------------: | :------: | :---------------------------------------------------------------------------------------------------------------------------------- |
| for.each |    `string`     |   `no`   | The name of the variable used to store the current item being enumerated.<br>Defaults to `item`.                                    |
| for.in   |    `string`     |  `yes`   | A [runtime expression](#runtime-expressions) used to get the collection to enumerate.                                               |
| for.at   |    `string`     |   `no`   | The name of the variable used to store the index of the current item being enumerated.<br>Defaults to `index`.                      |
| while    |    `string`     |   `no`   | A [runtime expression](#runtime-expressions) that represents the condition, if any, that must be met for the iteration to continue. |
| do       | [`task`](#task) |  `yes`   | The task to perform for each item in the collection.                                                                                |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  checkup:
    for:
      each: pet
      in: .pets
      at: index
    while: .vet != null
    do:
      checkForFleas:
        if: $pet.lastCheckup == null
        listen:
          to:
            one:
              with:
                type: com.fake.petclinic.pets.checkup.completed.v2
        output:
          to: '.pets + [{ "id": $pet.id }]'
```

#### Listen

Provides a mechanism for workflows to await and react to external events, enabling event-driven behavior within workflow systems.

##### Properties

| Name      |                           Type                            | Required | Description                                          |
| :-------- | :-------------------------------------------------------: | :------: | :--------------------------------------------------- |
| listen.to | [`eventConsumptionStrategy`](#event-consumption-strategy) |  `yes`   | Configures the event(s) the workflow must listen to. |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  callDoctor:
    listen:
      to:
        any:
          - with:
              type: com.fake-hospital.vitals.measurements.temperature
              data:
                temperature: ${ .temperature > 38 }
          - with:
              type: com.fake-hospital.vitals.measurements.bpm
              data:
                temperature: ${ .bpm < 60 or .bpm > 100 }
```

#### Raise

Intentionally triggers and propagates errors. By employing the "Raise" task, workflows can deliberately generate error conditions, allowing for explicit error handling and fault management strategies to be implemented.

##### Properties

| Name        |       Type        | Required | Description                 |
| :---------- | :---------------: | :------: | :-------------------------- |
| raise.error | [`error`](#error) |  `yes`   | Defines the error to raise. |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  processTicket:
    switch:
      highPriority:
        when: .ticket.priority == "high"
        then: escalateToManager
      mediumPriority:
        when: .ticket.priority == "medium"
        then: assignToSpecialist
      lowPriority:
        when: .ticket.priority == "low"
        then: resolveTicket
      default:
        then: raiseUndefinedPriorityError
  raiseUndefinedPriorityError:
    raise:
      error:
        type: https://fake.com/errors/tickets/undefined-priority
        status: 400
        title: Undefined Priority
  escalateToManager: { ... }
  assignToSpecialist: { ... }
  resolveTicket: { ... }
```

#### Run

Provides the capability to execute external [containers](#container-process), [shell commands](#shell-process), [scripts](#script-process), or [workflows](#workflow-process).

##### Properties

| Name          |               Type                | Required | Description                                                                                                          |
| :------------ | :-------------------------------: | :------: | :------------------------------------------------------------------------------------------------------------------- |
| run.container | [`container`](#container-process) |   `no`   | The definition of the container to run.<br>_Required if `script`, `shell` and `workflow` have not been set._         |
| run.script    |    [`script`](#script-process)    |   `no`   | The definition of the script to run.<br>_Required if `container`, `shell` and `workflow` have not been set._         |
| run.shell     |     [`shell`](#shell-process)     |   `no`   | The definition of the shell command to run.<br>_Required if `container`, `script` and `workflow` have not been set._ |
| run.workflow  |  [`workflow`](#workflow-process)  |   `no`   | The definition of the workflow to run.<br>_Required if `container`, `script` and `shell` have not been set._         |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  runContainer:
    run:
      container:
        image: fake-image

  runScript:
    run:
      script:
        language: js
        code: >
          Some cool multiline script

  runShell:
    run:
      shell:
        command: 'echo "Hello, ${ .user.name }"'

  runWorkflow:
    run:
      workflow:
        reference: another-one:0.1.0
        input: {}
```

##### Container Process

Enables the execution of external processes encapsulated within a containerized environment, allowing workflows to interact with and execute complex operations using containerized applications, scripts, or commands.

###### Properties

| Name        |   Type   | Required | Description                                                                                          |
| :---------- | :------: | :------: | :--------------------------------------------------------------------------------------------------- |
| image       | `string` |  `yes`   | The name of the container image to run                                                               |
| command     | `string` |   `no`   | The command, if any, to execute on the container                                                     |
| ports       |  `map`   |   `no`   | The container's port mappings, if any                                                                |
| volumes     |  `map`   |   `no`   | The container's volume mappings, if any                                                              |
| environment |  `map`   |   `no`   | A key/value mapping of the environment variables, if any, to use when running the configured process |

###### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  runContainer:
    run:
      container:
        image: fake-image
```

##### Script Process

Enables the execution of custom scripts or code within a workflow, empowering workflows to perform specialized logic, data processing, or integration tasks by executing user-defined scripts written in various programming languages.

###### Properties

| Name        |                  Type                  | Required | Description                                                                                          |
| :---------- | :------------------------------------: | :------: | :--------------------------------------------------------------------------------------------------- |
| language    |                `string`                |  `yes`   | The language of the script to run                                                                    |
| code        |                `string`                |   `no`   | The script's code.<br>_Required if `source` has not been set._                                       |
| source      | [externalResource](#external-resource) |   `no`   | The script's resource.<br>_Required if `code` has not been set._                                     |
| environment |                 `map`                  |   `no`   | A key/value mapping of the environment variables, if any, to use when running the configured process |

###### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  runScript:
    run:
      script:
        language: js
        code: >
          Some cool multiline script
```

##### Shell Process

Enables the execution of shell commands within a workflow, enabling workflows to interact with the underlying operating system and perform system-level operations, such as file manipulation, environment configuration, or system administration tasks.

###### Properties

| Name        |   Type   | Required | Description                                                                                          |
| :---------- | :------: | :------: | :--------------------------------------------------------------------------------------------------- |
| command     | `string` |  `yes`   | The shell command to run                                                                             |
| arguments   |  `map`   |   `no`   | A list of the arguments of the shell command to run                                                  |
| environment |  `map`   |   `no`   | A key/value mapping of the environment variables, if any, to use when running the configured process |

###### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  runShell:
    run:
      shell:
        command: 'echo "Hello, ${ .user.name }"'
```

##### Workflow Process

Enables the invocation and execution of nested workflows within a parent workflow, facilitating modularization, reusability, and abstraction of complex logic or business processes by encapsulating them into standalone workflow units.

###### Properties

| Name    |   Type   | Required | Description                                                                                                                                           |
| :------ | :------: | :------: | :---------------------------------------------------------------------------------------------------------------------------------------------------- |
| name    | `string` |  `yes`   | The name of the workflow to run                                                                                                                       |
| version | `string` |  `yes`   | The version of the workflow to run. Defaults to `latest`                                                                                              |
| input   |  `any`   |   `no`   | The data, if any, to pass as input to the workflow to execute. The value should be validated against the target workflow's input schema, if specified |

###### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  runShell:
    run:
      workflow:
        reference: another-one:0.1.0
        input:
          foo: bar
```

#### Set

A task used to set data.

##### Properties

| Name |   Type   | Required | Description                              |
| :--- | :------: | :------: | :--------------------------------------- |
| set  | `object` |  `yes`   | A name/value mapping of the data to set. |

##### Examples

```yaml
document:
  dsl: 1.0.0-alpha1
  namespace: default
  name: set
  version: "0.1.0"
do:
  initialize:
    set:
      shape: circle
      size: ${ .configuration.size }
      fill: ${ .configuration.fill }
```

#### Switch

Enables conditional branching within workflows, allowing them to dynamically select different paths based on specified conditions or criteria

##### Properties

| Name   |                Type                 | Required | Description                                |
| :----- | :---------------------------------: | :------: | :----------------------------------------- |
| switch | [`map[string, case]`](#switch-case) |  `yes`   | A name/value map of the cases to switch on |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  processOrder:
    switch:
      case1:
        when: .orderType == "electronic"
        then: processElectronicOrder
      case2:
        when: .orderType == "physical"
        then: processPhysicalOrder
      default:
        then: handleUnknownOrderType
  processElectronicOrder:
    execute:
      sequentially:
        validatePayment: { ... }
        fulfillOrder: { ... }
    then: exit
  processPhysicalOrder:
    execute:
      sequentially:
        checkInventory: { ... }
        packItems: { ... }
        scheduleShipping: { ... }
    then: exit
  handleUnknownOrderType:
    execute:
      sequentially:
        logWarning: { ... }
        notifyAdmin: { ... }
```

##### Switch Case

Defines a switch case, encompassing of a condition for matching and an associated action to execute upon a match.

| Name |                Type                | Required | Description                                                                                                                                                                                                                                  |
| :--- | :--------------------------------: | :------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| when |              `string`              |   `no`   | A runtime expression used to determine whether or not the case matches.<br>_If not set, the case will be matched by default if no other case match._<br>_Note that there can be only one default case, all others **MUST** set a condition._ |
| then | [`flowDirective`](#flow-directive) |  `yes`   | The flow directive to execute when the case matches.                                                                                                                                                                                         |

#### Try

Serves as a mechanism within workflows to handle errors gracefully, potentially retrying failed tasks before proceeding with alternate ones.

##### Properties

| Name  |       Type        | Required | Description                                            |
| :---- | :---------------: | :------: | :----------------------------------------------------- |
| try   |  [`task`](#task)  |  `yes`   | The task to perform.                                   |
| catch | [`catch`](#catch) |  `yes`   | Configures the errors to catch and how to handle them. |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  processOrder:
    try:
      call: http
      with:
        method: get
        endpoint: https://
    catch:
      errors:
        with:
          type: https://serverlessworkflow.io.io/dsl/errors/types/communication
          status: 503
      as: error
      retry:
        delay:
          seconds: 3
        backoff:
          exponential: {}
        limit:
          attempt:
            count: 5
```

##### Catch

Defines the configuration of a catch clause, which a concept used to catch errors.

###### Properties

| Name       |          Type           | Required | Description                                                                            |
| :--------- | :---------------------: | :------: | :------------------------------------------------------------------------------------- |
| errors     | [`errorFilter`](#retry) |   `no`   | The definition of the errors to catch                                                  |
| as         |        `string`         |   `no`   | The name of the runtime expression variable to save the error as. Defaults to 'error'. |
| when       |        `string`         |   `no`   | A runtime expression used to determine whether or not to catch the filtered error      |
| exceptWhen |        `string`         |   `no`   | A runtime expression used to determine whether or not to catch the filtered error      |
| retry      | [`retryPolicy`](#retry) |   `no`   | The retry policy to use, if any, when catching errors                                  |
| do         |     [`task`](#task)     |   `no`   | The definition of the task to run when catching an error                               |

#### Wait

Allows workflows to pause or delay their execution for a specified period of time.

##### Properties

| Name |                Type                 | Required | Description                                                                                      |
| :--- | :---------------------------------: | :------: | :----------------------------------------------------------------------------------------------- |
| wait | `string`<br>[`duration`](#duration) |  `yes`   | The amount of time to wait.<br>If a `string`, must be a valid [ISO 8601](#) duration expression. |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  delay10Seconds:
    wait:
      seconds: 10
```

### Flow Directive

Flow Directives are commands within a workflow that dictate its progression.

| Directive  | Description                                                                                                                                                                                   |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `continue` | Instructs the workflow to proceed with the next task in line. This action may conclude the execution of a particular workflow or branch if there are not task defined after the continue one. |
| `exit`     | Halts the current branch's execution, potentially terminating the entire workflow if the current task resides within the main branch.                                                         |
| `end`      | Provides a graceful conclusion to the workflow execution, signaling its completion explicitly.                                                                                                |

### External Resource

Defines an external resource.

#### Properties

| Property       |                Type                 | Required | Description                                                                                             |
| -------------- | :---------------------------------: | :------: | ------------------------------------------------------------------------------------------------------- |
| name           |              `string`               |   `no`   | The name, if any, of the defined resource.                                                              |
| uri            |              `string`               |  `yes`   | The URI at which to get the defined resource.                                                           |
| authentication | [`authentication`](#authentication) |   `no`   | The authentication policy, or the name of the authentication policy, to use when fecthing the resource. |

##### Examples

```yaml
name: sample-resource
uri: https://fake.com/resource/0123
authentication:
  basic:
    username: admin
    password: 1234
```

### Authentication

Defines the mechanism used to authenticate users and workflows attempting to access a service or a resource.

#### Properties

| Property    |                            Type                            | Required | Description                                                                                                               |
| ----------- | :--------------------------------------------------------: | :------: | ------------------------------------------------------------------------------------------------------------------------- |
| basic       |       [`basicAuthentication`](#basic-authentication)       |   `no`   | The `basic` authentication scheme to use, if any.<br>Required if no other property has been set, otherwise ignored.       |
| bearer      |      [`bearerAuthentication`](#bearer-authentication)      |   `no`   | The `bearer` authentication scheme to use, if any.<br>Required if no other property has been set, otherwise ignored.      |
| certificate | [`certificateAuthentication`](#certificate-authentication) |   `no`   | The `certificate` authentication scheme to use, if any.<br>Required if no other property has been set, otherwise ignored. |
| digest      |      [`digestAuthentication`](#digest-authentication)      |   `no`   | The `digest` authentication scheme to use, if any.<br>Required if no other property has been set, otherwise ignored.      |
| oauth2      |             [`oauth2`](#oauth2-authentication)             |   `no`   | The `oauth2` authentication scheme to use, if any.<br>Required if no other property has been set, otherwise ignored.      |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
use:
  secrets:
    usernamePasswordSecret: {}
  authentication:
    sampleBasicFromSecret:
      basic: usernamePasswordSecret
do:
  getMessages:
    call: http
    with:
      method: get
      endpoint:
        uri: https://secured.fake.com/sample
        authentication: sampleBasicFromSecret
```

#### Basic Authentication

Defines the fundamentals of a 'basic' authentication.

##### Properties

| Property |   Type   | Required | Description          |
| -------- | :------: | :------: | -------------------- |
| username | `string` |  `yes`   | The username to use. |
| password | `string` |  `yes`   | The password to use. |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
use:
  authentication:
    sampleBasic:
      basic:
        username: admin
        password: 123
do:
  getMessages:
    call: http
    with:
      method: get
      endpoint:
        uri: https://secured.fake.com/sample
        authentication: sampleBasic
```

#### Bearer Authentication

Defines the fundamentals of a 'bearer' authentication

##### Properties

| Property |   Type   | Required | Description              |
| -------- | :------: | :------: | ------------------------ |
| token    | `string` |  `yes`   | The bearer token to use. |

##### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
do:
  getMessages:
    call: http
    with:
      method: get
      endpoint:
        uri: https://secured.fake.com/sample
        authentication:
          bearer:
            token: ${ .user.token }
```

#### Certificate Authentication

#### Digest Authentication

#### OAUTH2 Authentication

Defines the fundamentals of an 'oauth2' authentication

##### Properties

| Property      |              Type              | Required | Description                                                                                               |
| ------------- | :----------------------------: | :------: | --------------------------------------------------------------------------------------------------------- |
| authority     |            `string`            |  `yes`   | The URI that references the OAuth2 authority to use.                                                      |
| grant         |            `string`            |  `yes`   | The grant type to use.                                                                                    |
| client.id     |            `string`            |  `yes`   | The client id to use.                                                                                     |
| client.secret |            `string`            |   `no`   | The client secret to use, if any.                                                                         |
| scopes        |           `string[]`           |   `no`   | The scopes, if any, to request the token for.                                                             |
| audiences     |           `string[]`           |   `no`   | The audiences, if any, to request the token for.                                                          |
| username      |            `string`            |   `no`   | The username to use. Used only if the grant type is `Password`.                                           |
| password      |            `string`            |   `no`   | The password to use. Used only if the grant type is `Password`.                                           |
| subject       | [`oauth2Token`](#oauth2-token) |   `no`   | The security token that represents the identity of the party on behalf of whom the request is being made. |
| actor         | [`oauth2Token`](#oauth2-token) |   `no`   | The security token that represents the identity of the acting party.                                      |

##### Examples

```yaml
document:
  dsl: '1.0.0-alpha1'
  namespace: test
  name: sample-workflow
  version: '0.1.0'
do:
  getMessages:
    call: http
    with:
      method: get
      endpoint:
        uri: https://secured.fake.com/sample
        authentication:
          oauth2:
            authority: http://keycloak/realms/fake-authority/.well-known/openid-configuration
            grant: client-credentials
            client:
              id: workflow-runtime
              secret: **********
            scopes: [ api ]
            audiences: [ runtime ]
```

##### OAUTH2 Token

Represents the definition of an OAUTH2 token

###### Properties

| Property |   Type   | Required | Description                        |
| -------- | :------: | :------: | ---------------------------------- |
| token    | `string` |  `yes`   | The security token to use to use.  |
| type     | `string` |  `yes`   | The type of security token to use. |

### Extension

Holds the definition for extending functionality, providing configuration options for how an extension extends and interacts with other components.

Extensions enable the execution of tasks prior to those they extend, offering the flexibility to potentially bypass the extended task entirely using an [`exit` workflow directive](#flow-directive).

#### Properties

| Property |      Type       | Required | Description                                                                                                                                                             |
| -------- | :-------------: | :------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| extend   |    `string`     |  `yes`   | The type of task to extend<br>Supported values are: `call`, `composite`, `emit`, `extension`, `for`, `listen`, `raise`, `run`, `set`, `switch`, `try`, `wait` and `all` |
| when     |    `string`     |   `no`   | A runtime expression used to determine whether or not the extension should apply in the specified context                                                               |
| before   | [`task`](#task) |   `no`   | The task to execute, if any, before the extended task                                                                                                                   |
| after    | [`task`](#task) |   `no`   | The task to execute, if any, after the extended task                                                                                                                    |

#### Examples

_Perform logging before and after any non-extension task is run:_

```yaml
document:
  dsl: '1.0.0-alpha1'
  namespace: test
  name: sample-workflow
  version: '0.1.0'
use:
  extensions:
    logging:
      extend: all
      before:
        call: http
        with:
          method: post
          endpoint: https://fake.log.collector.com
          body:
            message: "${ \"Executing task '\($task.reference)'...\" }"
      after:
        call: http
        with:
          method: post
          endpoint: https://fake.log.collector.com
          body:
            message: "${ \"Executed task '\($task.reference)'...\" }"
do:
  get:
    call: http
    with:
      method: get
      endpoint: https://fake.com/sample
```

_Intercept HTTP calls to 'https://mocked.service.com' and mock its response:_

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: test
  name: sample-workflow
  version: "0.1.0"
use:
  extensions:
    mockService:
      extend: http
      when: ($task.with.uri != null and ($task.with.uri | startswith("https://mocked.service.com"))) or ($task.with.endpoint.uri != null and ($task.with.endpoint.uri | startswith("https://mocked.service.com")))
      before:
        set:
          statusCode: 200
          headers:
            Content-Type: application/json
          content:
            foo:
              bar: baz
        then: exit #using this, we indicate to the workflow we want to exit the extended task, thus just returning what we injected
do:
  get:
    call: http
    with:
      method: get
      endpoint: https://fake.com/sample
```

### Error

Defines the [Problem Details RFC](https://datatracker.ietf.org/doc/html/rfc7807) compliant description of an error.

#### Properties

| Property |   Type    | Required | Description                                                                                                                                                                                                                                                                                                                                                                           |
| -------- | :-------: | :------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| type     | `string`  |  `yes`   | A URI reference that identifies the [`error`](#error) type. <br><u>For cross-compatibility concerns, it is strongly recommended to use [Standard Error Types](#standard-error-types) whenever possible.<u><br><u>Runtimes **MUST** ensure that the property has been set when raising or escalating the [`error`](#error).<u>                                                         |
| status   | `integer` |  `yes`   | The status code generated by the origin for this occurrence of the [`error`](#error).<br><u>For cross-compatibility concerns, it is strongly recommended to use [HTTP Status Codes](https://datatracker.ietf.org/doc/html/rfc7231#section-6) whenever possible.<u><br><u>Runtimes **MUST** ensure that the property has been set when raising or escalating the [`error`](#error).<u> |
| instance | `string`  |  `yes`   | A [JSON Pointer](https://datatracker.ietf.org/doc/html/rfc6901) used to reference the component the [`error`](#error) originates from.<br><u>Runtimes **MUST** set the property when raising or escalating the [`error`](#error). Otherwise ignore.<u>                                                                                                                                |
| title    | `string`  |   `no`   | A short, human-readable summary of the [`error`](#error).                                                                                                                                                                                                                                                                                                                             |
| detail   | `string`  |   `no`   | A human-readable explanation specific to this occurrence of the [`error`](#error).                                                                                                                                                                                                                                                                                                    |

#### Examples

```yaml
type: https://https://serverlessworkflow.io/spec/1.0.0/errors/communication
title: Service Not Available
status: 503
```

#### Standard Error Types

Standard error types serve the purpose of categorizing errors consistently across different runtimes, facilitating seamless migration from one runtime environment to another.

| Type                                                                        | Status¹ | Description                                                                                                                                                                                                                                                                                                                       |
| --------------------------------------------------------------------------- | :-----: | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [https://https://serverlessworkflow.io/spec/1.0.0/errors/configuration](#)  |  `400`  | Errors resulting from incorrect or invalid configuration settings, such as missing or misconfigured environment variables, incorrect parameter values, or configuration file errors.                                                                                                                                              |
| [https://https://serverlessworkflow.io/spec/1.0.0/errors/validation](#)     |  `400`  | Errors arising from validation processes, such as validation of input data, schema validation failures, or validation constraints not being met. These errors indicate that the provided data or configuration does not adhere to the expected format or requirements specified by the workflow.                                  |
| [https://https://serverlessworkflow.io/spec/1.0.0/errors/expression](#)     |  `400`  | Errors occurring during the evaluation of runtime expressions, such as invalid syntax or unsupported operations.                                                                                                                                                                                                                  |
| [https://https://serverlessworkflow.io/spec/1.0.0/errors/authentication](#) |  `401`  | Errors related to authentication failures.                                                                                                                                                                                                                                                                                        |
| [https://https://serverlessworkflow.io/spec/1.0.0/errors/authorization](#)  |  `403`  | Errors related to unauthorized access attempts or insufficient permissions to perform certain actions within the workflow.                                                                                                                                                                                                        |
| [https://https://serverlessworkflow.io/spec/1.0.0/errors/timeout](#)        |  `408`  | Errors caused by timeouts during the execution of tasks or during interactions with external services.                                                                                                                                                                                                                            |
| [https://https://serverlessworkflow.io/spec/1.0.0/errors/communication](#)  |  `500`  | Errors encountered while communicating with external services, including network errors, service unavailable, or invalid responses.                                                                                                                                                                                               |
| [https://https://serverlessworkflow.io/spec/1.0.0/errors/runtime](#)        |  `500`  | Errors occurring during the runtime execution of a workflow, including unexpected exceptions, errors related to resource allocation, or failures in handling workflow tasks. These errors typically occur during the actual execution of workflow components and may require runtime-specific handling and resolution strategies. |

¹ _Default value. The `status code` that best describe the error should always be used._

### Event Consumption Strategy

Represents the configuration of an event consumption strategy.

#### Properties

| Property |               Type               | Required | Description                                                                                                                                  |
| -------- | :------------------------------: | :------: | -------------------------------------------------------------------------------------------------------------------------------------------- |
| all      | [`eventFilter[]`](#event-filter) |   `no`   | Configures the workflow to wait for all defined events before resuming execution.<br>_Required if `any` and `one` have not been set._        |
| any      | [`eventFilter[]`](#event-filter) |   `no`   | Configures the workflow to wait for any of the defined events before resuming execution.<br>_Required if `all` and `one` have not been set._ |
| one      |  [`eventFilter`](#event-filter)  |   `no`   | Configures the workflow to wait for the defined event before resuming execution.<br>_Required if `all` and `any` have not been set._         |

### Event Filter

An event filter is a mechanism used to selectively process or handle events based on predefined criteria, such as event type, source, or specific attributes.

#### Properties

| Property  |                    Type                    | Required | Description                                                                                                                    |
| --------- | :----------------------------------------: | :------: | ------------------------------------------------------------------------------------------------------------------------------ |
| with      |                  `object`                  |  `yes`   | A name/value mapping of the attributes filtered events must define. Supports both regular expressions and runtime expressions. |
| correlate | [`map[string, correlation]`](#correlation) |   `no`   | A name/definition mapping of the correlations to attempt when filtering events.                                                |

### Correlation

A correlation is a link between events and data, established by mapping event attributes to specific data attributes, allowing for coordinated processing or handling based on event characteristics.

#### Properties

| Property |   Type   | Required | Description                                                                                                                                                                                                                |
| -------- | :------: | :------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| from     | `string` |  `yes`   | A runtime expression used to extract the correlation value from the filtered event.                                                                                                                                        |
| expect   | `string` |   `no`   | A constant or a runtime expression, if any, used to determine whether or not the extracted correlation value matches expectations.<br>If not set, the first extracted value will be used as the correlation's expectation. |

### Retry

The Retry is a fundamental concept in the Serverless Workflow DSL, used to define the strategy for retrying a failed task when an error is encountered during execution. This policy provides developers with fine-grained control over how and when to retry failed tasks, enabling robust error handling and fault tolerance within workflows.

#### Properties

| Property   |          Type           | Required | Description                                                                                             |
| ---------- | :---------------------: | :------: | ------------------------------------------------------------------------------------------------------- |
| when       |        `string`         |   `no`   | A a runtime expression used to determine whether or not to retry running the task, in a given context.  |
| exceptWhen |        `string`         |   `no`   | A runtime expression used to determine whether or not to retry running the task, in a given context.    |
| limit      | [`retry`](#retry-limit) |   `no`   | The limits, if any, to impose to the retry policy.                                                      |
| backoff    |  [`backoff`](#backoff)  |   `no`   | The backoff strategy to use, if any.                                                                    |
| jitter     |   [`jitter`](#jitter)   |   `no`   | The parameters, if any, that control the randomness or variability of the delay between retry attempts. |

#### Retry Limit

The definition of a retry policy.

| Property         |          Type           | Required | Description                                                       |
| ---------------- | :---------------------: | :------: | ----------------------------------------------------------------- |
| attempt.count    |        `integer`        |   `no`   | The maximum attempts count.                                       |
| attempt.duration | [`duration`](#duration) |   `no`   | The duration limit, if any, for all retry attempts.               |
| duration         | [`duration`](#duration) |   `no`   | The maximum duration, if any, during which to retry a given task. |

#### Backoff

The definition of a retry backoff strategy.

| Property    |   Type   | Required | Description                                                                                                                        |
| ----------- | :------: | :------: | ---------------------------------------------------------------------------------------------------------------------------------- |
| constant    | `object` |   `no`   | The definition of the constant backoff to use, if any.<br>_Required if `exponential` and `linear` are not set, otherwise ignored._ |
| exponential | `object` |   `no`   | The definition of the exponential backoff to use, if any.<br>_Required if `constant` and `linear` are not set, otherwise ignored._ |
| linear      | `object` |   `no`   | The definition of the linear backoff to use, if any.<br>_Required if `constant` and `exponential` are not set, otherwise ignored._ |

#### Jitter

Represents the definition of the parameters that control the randomness or variability of a delay, typically between retry attempts

| Property |          Type           | Required | Description                               |
| -------- | :---------------------: | :------: | ----------------------------------------- |
| from     | [`duration`](#duration) |  `yes`   | The minimum duration of the jitter range. |
| to       | [`duration`](#duration) |  `yes`   | The maximum duration of the jitter range. |

#### Examples

```yaml

```

### Input

Documents the structure - and optionally configures the filtering of - workflow/task input data.

It's crucial for authors to document the schema of input data whenever feasible. This documentation empowers consuming applications to provide contextual auto-suggestions when handling runtime expressions.

When set, runtimes must validate input data against the defined schema, unless defined otherwise.

#### Properties

| Property |         Type         | Required | Description                                                                                                                                                                 |
| -------- | :------------------: | :------: | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| schema   | [`schema`](#schema)  |   `no`   | The [`schema`](#schema) used to describe and validate input data.<br>_Even though the schema is not required, it is strongly encouraged to document it, whenever feasible._ |
| from     | `string`<br>`object` |   `no`   | A [runtime expression](#runtime-expressions), if any, used to filter and/or mutate the workflow/task input.                                                                 |

#### Examples

```yaml
schema:
  format: json
  document:
    type: object
    properties:
      petId:
        type: string
    required: [petId]
from: .order.pet
```

### Output

Documents the structure - and optionally configures the filtering of - workflow/task output data.

It's crucial for authors to document the schema of output data whenever feasible. This documentation empowers consuming applications to provide contextual auto-suggestions when handling runtime expressions.

When set, runtimes must validate output data against the defined schema, unless defined otherwise.

#### Properties

| Property |         Type         | Required | Description                                                                                                                                                                  |
| -------- | :------------------: | :------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| schema   | [`schema`](#schema)  |   `no`   | The [`schema`](#schema) used to describe and validate output data.<br>_Even though the schema is not required, it is strongly encouraged to document it, whenever feasible._ |
| from     | `string`<br>`object` |   `no`   | A [runtime expression](#runtime-expressions), if any, used to filter and/or mutate the workflow/task output.                                                                 |
| to       | `string`<br>`object` |   `no`   | A [runtime expression](#runtime-expressions), if any, used to update the context, using both output and context data.                                                        |

#### Examples

```yaml
schema:
  format: json
  document:
    type: object
    properties:
      petId:
        type: string
    required: [petId]
from:
  petId: "${ .pet.id }"
to: ".petList += [ . ]"
```

### Schema

Describes a data schema.

#### Properties

| Property |                   Type                   | Required | Description                                                                                                                     |
| -------- | :--------------------------------------: | :------: | ------------------------------------------------------------------------------------------------------------------------------- |
| format   |                 `string`                 |  `yes`   | The schema format.<br>_Supported values are:_<br>_- `json`, which indicates the [JsonSchema](https://json-schema.org/) format._ |
| document |                 `object`                 |   `no`   | The inline schema document.<br>_Required if `resource` has not been set, otherwise ignored._                                    |
| resource | [`externalResource`](#external-resource) |   `no`   | The schema external resource.<br>_Required if `document` has not been set, otherwise ignored._                                  |

#### Examples

_Example of an inline JsonSchema:_

```yaml
format: json
document:
  type: object
  properties:
    id:
      type: string
    firstName:
      type: string
    lastName:
      type: string
  required: [id, firstName, lastName]
```

_Example of a JsonSchema based on an external resource:_

```yaml
format: json
resource:
  uri: https://test.com/fake/schema/json/document.json
```

### Timeout

Defines a workflow or task timeout.

#### Properties

| Property |          Type           | Required | Description                                              |
| -------- | :---------------------: | :------: | -------------------------------------------------------- |
| after    | [`duration`](#duration) |  `yes`   | The duration after which the workflow or task times out. |

#### Examples

```yaml
document:
  dsl: "1.0.0-alpha1"
  namespace: default
  name: sample
  version: "0.1.0"
do:
  waitFor60Seconds:
    wait:
      seconds: 60
timeout:
  after:
    seconds: 30
```

### Duration

Defines a duration.

#### Properties

| Property     |   Type    | Required | Description                     |
| ------------ | :-------: | :------: | ------------------------------- |
| Days         | `integer` |   `no`   | Number of days, if any.         |
| Hours        | `integer` |   `no`   | Number of hours, if any.        |
| Minutes      | `integer` |   `no`   | Number of minutes, if any.      |
| Seconds      | `integer` |   `no`   | Number of seconds, if any.      |
| Milliseconds | `integer` |   `no`   | Number of milliseconds, if any. |

#### Examples

_Example of a duration of 2 hours, 15 minutes and 30 seconds:_

```yaml
hours: 2
minutes: 15
seconds: 30
```

### HTTP Response

Describes an HTTP response.

#### Properties

| Property   |            Type            | Required | Description                                                                                                                                                                                                              |
| ---------- | :------------------------: | :------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| request    | [`request`](#http-request) |  `yes`   | The HTTP request associated with the HTTP response.                                                                                                                                                                      |
| statusCode |         `integer`          |  `yes`   | The HTTP response status code.                                                                                                                                                                                           |
| headers    |   `map[string, string]`    |   `no`   | The HTTP response headers, if any.                                                                                                                                                                                       |
| content    |           `any`            |   `no`   | The HTTP response content, if any.<br>_If the request's content type is one of the following, should contain the deserialized response content. Otherwise, should contain the base-64 encoded response content, if any._ |

#### Examples

```yaml
request:
  method: get
  uri: https://petstore.swagger.io/v2/pet/1
  headers:
    Content-Type: application/json
headers:
  Content-Type: application/json
statusCode: 200
content:
  id: 1
  name: milou
  status: pending
```

### HTTP Request

Describes an HTTP request.

#### Properties

| Property |         Type          | Required | Description                       |
| -------- | :-------------------: | :------: | --------------------------------- |
| method   |       `string`        |  `yes`   | The request's method.             |
| uri      |         `uri`         |  `yes`   | The request's URI.                |
| headers  | `map[string, string]` |   `no`   | The HTTP request headers, if any. |

#### Examples

```yaml
method: get
uri: https://petstore.swagger.io/v2/pet/1
headers:
  Content-Type: application/json
```
