# CRM for Organizations

A **multi-user Customer Relationship Management (CRM) platform** built with Django that allows organizations to manage leads, assign sales agents, and track sales pipeline stages.

This project demonstrates how to design a **role-based, multi-tenant business application** where organizations manage teams and teams manage leads.

It reflects the architecture commonly used in **real SaaS CRM systems**.

---

# Project Preview

## Dashboard

![CRM Dashboard](docs/screenshots/dashboard.png)

---

## Lead Management

![Lead Management](docs/screenshots/leads.png)

---

## Agent Management

![Agent Management](docs/screenshots/agents.png)

---

## User Management

![User Management](docs/screenshots/users.png)

---

# Core Features

## Role-Based Access System

The system defines two user roles.

### Organizer

Organizers act as administrators of the organization.

Capabilities:

- create agents
- create and manage leads
- assign leads to agents
- manage lead categories
- view all organization data

---

### Agent

Agents are responsible for working on assigned leads.

Capabilities:

- view assigned leads
- track leads through categories

Agents cannot create agents or manage organization configuration.

---

# Multi-Tenant Architecture

The system isolates data per organization.

Every important object belongs to an organization:

- Leads
- Agents
- Categories

This ensures:

- organizations cannot see each other's data
- agents only access leads assigned to them
- the platform can scale to support multiple organizations

This pattern is widely used in **SaaS platforms**.

---

# Lead Management System

Leads represent potential customers.

Each lead contains:

- first name
- last name
- age
- assigned agent
- category
- organization

Leads can be created, edited, deleted, and assigned to agents.

---

# Lead Assignment Workflow

Leads are distributed across agents through an assignment workflow.

Typical flow:

1. Organizer creates a lead
2. Lead appears as **unassigned**
3. Organizer assigns the lead to an agent
4. The agent can now work on that lead

This models how real sales teams distribute incoming prospects.

---

# Lead Categorization

Leads can be organized into pipeline stages such as:

- New
- Contacted
- Converted
- Unconverted

Categories allow teams to track **sales progress and lead status**.

---

# Agent Management

Organizers can manage their sales team directly from the system.

They can:

- create agents
- update agent information
- remove agents
- monitor agent assignments

When an agent is created:

- a user account is generated
- the agent is linked to the organization
- an invitation email is sent

---

# Authentication System

The project uses Django authentication with a **custom user model**.

Additional attributes define user roles:

- `is_organizer`
- `is_agent`

Each user automatically receives a **UserProfile** representing their organization.  
This is handled using a **Django signal**.

---

# Access Control

Access control is implemented through:

- login protection
- role-based permission checks
- organization-scoped query filtering

This ensures users cannot access data outside their organization.

---

