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

