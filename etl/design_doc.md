[ETL Tool] Project Refactor

# Background

[ETL Tool], the defacto ETL utility used to import customer data into a new database instance of
MadeUp Co., was initially designed to work with only ONE database technology and was meant to be run
ONCE against said database. However, this utility now has the need to function against multiple
database technologies and be run in a repeatable fashion. Some inroads have been made to make this
utility work with additional databases, but the time has come to expand this prototype to a more
scalable solution.

# Goals

The goal of this document will be to define a meaningful path forward for the [ETL Tool] project so that
we can reduce onboarding time for our customers and development time for engineering team with a
more scalable solution that will support MANY database technologies and run MANY times. This “final
phase” can then be used to work backward and define a phased roll out of the updates required.

# Initial Assessment

## Workflow Summary

![current-design](architecture-design-diagram/etl/current-design.png)

- A defined exporter class is initialized with a block of Vendor X record IDs.
- The exporter queries the DB for the records and any related entities related to them.
- Export Phases
  - The exporter enqueues a delayed job with the individual entity hash.
  - The importer builds the entity and any cascading records
  - The exporter enqueues a delayed job with the individual record hash.
  - The importer builds the record and any cascading records

## Technical Debt

- This project has to be synchronized with Core DB and Model validations (duplication)
- All transformation are run on a single server and single thread (single threaded operation)
- Throughput is limited by the server running the task
- Dividing data into manageable sections is a manual process
- Each exporter is custom-tailored to the Vendor (does not ascribe to a contract)
- Extraction and Transformation are handled by a single class (overloaded)
- Low amount of encapsulation among classes (no unit testing does not ascribe to TDD)
- Multiple DB connections/technologies to maintain/manage (manageable but not ideal)

# Options For Improvement

To assess our ability and resources required to accomplish a refactor of this tool the following
options have been provided. Regardless of the technology path chosen the solution must adhere to
these basic requirements.

## Requirements

- Must remove duplication created by keeping ETL project in sync with [Main Project project
- Must be thread safe to process more transformations in less time
- Must be testable (unit, integration and end-to-end)

|             | Option 1                       | Option 2                               |
| ----------- | ------------------------------ | -------------------------------------- |
| Description | Kafka Backbone (see diagram)   | REST API (see diagram)                 |
| Pros        | removes duplication            | removes duplication                    |
|             | leverages existing export code | leverages existing export code         |
|             | creates thread-safe export     | creates thread-safe export             |
|             | asynchronous loaded            | asynchronous loader                    |
|             | ERD is owned by [Main Project] | ERD is owned by [Main Project]         |
|             | Built in redundancy (Kafka)    | Creates/Exposes a RESTful API          |
| Cons        | Dev-Ops overhead               | Bottlenecking dependent on replication |
| Cost        | **Large**                      | **Medium**                             |

## Kafka Diagram

![kafka-design](architecture-design-diagram/etl/kafka-design.png)

## REST Diagram

![rest-design](architecture-design-diagram/etl/rest-design.png)

# Action Items

- [ ] Create common Transformer interface (break up transform and load concerns in Vendor models)
- [ ] Prioritize the FanOut worker to asynchronously load/transform data
- [ ] Prioritize a standard CLI, Console, or RESTful interface to run the utility
- [ ] Encapsulate and increase test coverage with all new code PRs
