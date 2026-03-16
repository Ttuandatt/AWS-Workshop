+++
title = "Warehouse Management Desktop Application"
description = "A desktop application for streamlining inventory tracking, product management, and stock movements. Built with Java SE following 3-Tier Architecture with DAO and DTO patterns."
weight = 4
chapter = false
layout = "project-detail"
tags = ["Java SE", "Java Swing", "MySQL", "JDBC", "3-Tier Architecture", "Team Leader"]
+++

## Overview

**Warehouse Management** is a desktop application I developed as **Team Leader** (team of 3) to streamline inventory tracking, product management, and stock movements. The system follows the **3-Tier Architecture** to ensure separation of concerns and maintainability.

🔗 **GitHub**: [Warehouse_Management](https://github.com/Ttuandatt/Warehouse_Management.git)

📅 **Timeline**: Mar 2024 – Sep 2024

---

## Architecture

The application follows a strict **3-Layer Architecture**:

```
┌──────────────────────────┐
│      GUI Layer           │  ← Java Swing / JavaFX
│  (Presentation / View)   │
└───────────┬──────────────┘
            │
┌───────────▼──────────────┐
│      BUS Layer           │  ← Business Logic
│  (Business / Service)    │
└───────────┬──────────────┘
            │
┌───────────▼──────────────┐
│      DAO Layer           │  ← Data Access Objects
│  (Data Access / JDBC)    │     + DTO Pattern
└───────────┬──────────────┘
            │
┌───────────▼──────────────┐
│      MySQL Database      │
└──────────────────────────┘
```

---

## Key Responsibilities

### Leadership & Architecture
- Led the team in designing the system architecture based on **DAO** (Data Access Object) and **DTO** (Data Transfer Object) patterns to decouple business logic from data access

### Database Implementation
- Engineered the Data Access Layer using **JDBC**
- Wrote raw SQL queries via `PreparedStatement` to optimize performance and **prevent SQL Injection attacks**

### Module Development
- Implemented the secure **Authentication module** (Login/Logout)
- Built core **Inventory Management** features (Product CRUD, Stock In/Out)

### UI/UX Design
- Designed the main application layout using **Java Swing/JavaFX**
- Ensured an intuitive and user-friendly experience for warehouse staff

---

## Technology Stack

- **Core**: Java SE (OOP), Java Swing / JavaFX
- **Architecture**: 3-Layer (GUI → BUS → DAO), DTO Pattern
- **Database**: MySQL, JDBC (Native Driver)
- **Tools**: IntelliJ IDEA, Visual Paradigm

---

## Outcomes

- Delivered a fully functional desktop application with complete inventory management capabilities
- Applied enterprise-grade design patterns (DAO, DTO) ensuring maintainability and scalability
- Gained leadership experience managing a 3-person team through full development lifecycle
