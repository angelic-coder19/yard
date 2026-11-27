# 0001: Postgres as source of truth
Status: Accepted · Date: 2026-10-31
## Context
Yard stores users, events, organizations, schools, sources, RSVPs. One constraint is that each user can have one RSVP for one specific event. Each user, organization, and event belong to a school. In further iterations where yard expands to outside buisnesses, the organization will be tied to a school even though it is not at that campus because that is school that organization's event will be pushed to. 

## Decision

PostgreSQl - It is best for structured data where relationships between different data fields are well defined. 
Follows the ACID principle: Atomicity - one transaction failiure must not cause any partial changes, a roll back to previous state to protect data
                            Consistency - one transaction must follow the schema and rules must apply to all parts of database
                            Isolation - transaction happening at the same time must run without conflicting with each other
                            Durability - a transactions actions must be permanent until another transaction changes the state of the databse

## Alternatives considered
I considered using MongoDB, DynamoDB for everything, and MySQL. 
MongoDB stores information as flexible json objects, DynmoDB is used for fast lookups has no easy joins because it is a single organinzed table, both follow BASE (Basic Availability, Soft State, Eventual Consistency). 
MySQL could perform the same function too because it follows ACID, is structured. However it is a file based database that serverless platforms do not support it (been here before, I had to perform a whole database migration because i did not do my research once!)

## Consequences
Lookups will be predictable, joins will be quick because postgres allows indeces to be created on the core columns that are looked up the most. Data will stay consistent and accurate because of ACID. Since joins are possible, getting information on the number of RSVPS for a specific club have been made will be quick and reliable compared to other options.
