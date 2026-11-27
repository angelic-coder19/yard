# 0001: Postgres as source of truth
Status: Accepted · Date: 2026-10-31
## Context
Yard stores users, events, organizations, schools, sources, RSVPs. One constraint is that each user can have one RSVP for one specific event. Each user, organization, and event belong to a school. In further iterations where yard expands to outside buisnesses, the organization will be tied to a school even though it is not at that campus because that is school that organization's event will be pushed to. 

## Decision
*Postgres is the source of truth for all core entities and Redis is a derived cache.*

PostgreSQl - It is best for structured data where relationships between different data fields are well defined. 
Follows the ACID principle: 
- Atomicity - one transaction failiure must not cause any partial changes, a roll back to previous state to protect data. This means an event cannot be partially added to the database or deleted. 
- Consistency - one transaction must follow the schema and leave all the established constraints true. Each user is restricted to one RSVP per event without exceptions. 
- Isolation - transaction happening at the same time must run without conflicting or interfering with each other. Is a two or more users RSVP the same event at the same time, the database should register each user uniquely for an accurate number of RSVPs.
- Durability - a transaction actions must be persist permanently even after power loss. A user should never have to loose their saved event/RSVP neither should an event that has been added to the list of events ever get lost because the server went down or lost power.



## Alternatives considered
I considered using MongoDB, DynamoDB for everything, and MySQL. 
- MongoDB stores information as flexible json objects and follows BASE (Basic Availability, Soft State, Eventual Consistency). 
- DynamoDB's access pattern is centered around the idea of knowing exactly what information will the appliation will need and designing a database around that. For Yard, not all the queries are known yet, so this does not fit this specific use case.
- MySQL could perform the same function too because it follows ACID, is structured. However, data from the scraped event will be messy but still useful MySQL does not provide a clean and fast solution that allows me to make that unstructured info useful. Postgres supports JSONB which can store that unstructured info in binary and can make it quickly searchable by keyword. 

## Consequences
### Upsides
- Lookups will be predictable
- Data will stay consistent and accurate because of ACID. Since joins are possible, getting information on the number of RSVPS for a specific club have been made will be quick and reliable compared to other options.
- JSONB allows for more flexibility because the unstructured part of the info extracted by the AI like zoom links, or descriptions can be added here are are quickly searchable.

### Potential Downsides
- A serverless connection will exhaust postgress' resources so a connection pool has to be created for a number of queries before being written to the database.
- Because Postgres is a structured database, future schema changes will require a whole database migration 
- We don't want every user's visit to Yard to open a read query, that is why the cache exists to save on trips and requests made to the postgres server
