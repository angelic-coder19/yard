# 0001: Postgres as source of truth
Status: Accepted · Date: 2026-11-27
## Context
Yard stores users, events, organizations, schools, sources (future: ingest), RSVPs. One constraint is that each user can have one RSVP for one specific event. Each user, organization, and event belong to a school. In further iterations where yard expands to outside businesses, the organization will be tied to a school even though it is not at that campus because that is the school that organization's event will be pushed to. 

### Access Pattern
hot path: feed for one school, upcoming events by starts_at, 20 at a time
*The hot path is the query that will run most often when the application runs*

## Decision
*Postgres is the source of truth for all core entities and Redis is a derived cache.*

Postgres - It fits Yard's structured relational data. 
Follows the ACID principle: 
- Atomicity - one transaction failure must not cause any partial changes, a roll back to previous state to protect data. This means an event cannot be partially added to the database or deleted. 
- Consistency - one transaction must follow the schema and leave all the established constraints true. Each user is restricted to one RSVP per event without exceptions. Additionally, if a user double-taps the RSVP button only one should go through. UNIQUE (user_id, event_id) ensures that only one row is created.
- Isolation - transaction happening at the same time must run without conflicting or interfering with each other. For example, a student loading the feed while an org edits an event sees the old row or the new row, never a half-written one.
- Durability - a committed transaction's actions must persist permanently even after power loss. A user should never have to lose their saved event/RSVP neither should an event that has been added to the list of events ever get lost because the server went down or lost power.

## Alternatives considered
I considered using MongoDB, DynamoDB for everything, and MySQL. 
- MongoDB stores information as flexible json objects. The data in Yard is relational, MongoDB's json structure does not cleanly enforce that relational structure the way postgres does with foreign keys
- DynamoDB's access pattern is centered around the idea of knowing exactly what information the application will need and designing a database around that. For Yard, not all the queries are known yet, so this does not fit this specific use case. Revisit for the ingest/dedupe path (ADR 0003).
- MySQL could perform the same function too because it follows ACID, is structured. Data from the AI Scraper will be messy but useful. While mysql supports unstructured data, Postgres supports JSONB with Generalized Inverted Index (GIN) which makes key value pairs for the whole document. Mysql would have to index specific paths one at a time with a generated column

## Consequences
### Upsides
- The relational nature of the entities in the application will be leveraged to enforce relationships and get additional information in future based on the structure. To illustrate we can get the number of RSVPs for a specific organization at a school using a join, the number of rows can be obtained cleanly. Data will stay consistent and accurate because of ACID. 
- JSONB allows for more flexibility because the unstructured part of the info extracted by the AI like zoom links, or ticket URLs can be added here are quickly searchable. JSONB will be added as an additional column in the database events table to contain information that did not cleanly fit into the defined structured fields. 

### Potential Downsides
- An individual serverless connection for each client request will exhaust Postgres's resources (specifically the maximum number of connections that can be opened at once to a postgres database) so a connection pool has to be created to reduce the number of individual database connections the client makes with the database server.
- Because Postgres is a structured database, future schema changes will require small table migrations that may lock some tables while they run
- We don't want every user's visit to Yard to open a read query, that is why the cache exists to save on trips and requests made to the postgres server
