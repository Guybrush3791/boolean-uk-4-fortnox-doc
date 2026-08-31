# Data Modelling

## Slides
[[Data Modelling.pdf|Slides link]]

## Learning Objectives
- Understand **what data modelling is**.
- Follow the **data-modelling process** step by step.
- Distinguish the **three stages** of data modelling.
- Learn how to create an **Entity Relationship (ER) Model**.

## Introduction
Data modelling is the practice of planning and illustrating how information will be stored so it meets business needs. A good model shows:

- The kinds of data collected.
- Relationships among that data.
- How data can be grouped, formatted and attributed.

Done well, data modelling:

- Reduces development errors.
- Keeps documentation and design consistent.
- Boosts application and database performance.
- Simplifies data mapping across the organisation.
- Improves communication between developers and business-intelligence teams.

## Data-Modelling Process
1. **Identify entities:** list every thing, event or concept to be represented.
2. **Identify key properties:** note the unique attributes that distinguish each entity.
3. **Identify relationships:** draft how entities relate to one another.
4. **Map attributes completely:** ensure every required attribute is attached to the correct entity.
5. **Assign keys and decide normalisation:** choose primary and foreign keys and the degree of normalisation needed to avoid duplicate data.

## Stages of Data Modelling

### Conceptual model
This is the big-picture view of what the system contains, how it is organised and which business rules apply. The notation remains simple.

![[Data Modelling - Conceptual|500]]

### Logical model
This adds attributes, data types, lengths and all relationships while remaining independent of any specific technology.

![[Data Modelling - Logical|700]]

### Physical model
This is a production-ready blueprint of tables, primary keys and foreign keys that maintain relationships.

![[Data Modelling - Physical|900]]

## Creating an Entity Relationship Model
ER diagrams formally map entities and the relationships between them. Tools such as [draw.io](https://app.diagrams.net/) make it easy to build these visual models before implementation.

---

# Links
![[Lessons/2 - Java Back-end/Day 10/__blocks/Links]]
