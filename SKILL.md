---
name: database-designer-lite
description: Design databases for personal-level projects based on requirements. Generates a markdown design document and executable SQL, saving them to a local file. Follows basic normalization and extensibility rules, with logical deletion for business data and physical deletion for temporary data. Trigger when the user asks for database design, schema creation, or SQL generation for a personal project.
---

# Personal Database Designer

This skill helps users design databases for personal-level projects. It balances simplicity with good design practices, ensuring the database is not over-engineered (avoiding complex enterprise structures) but is robust, normalized, and extensible.

## Input

The user will provide:

1. A requirement description, project name, or design document.
2. The target database type (e.g., MySQL, PostgreSQL, SQLite). If not specified, ask or default to a common choice like MySQL or PostgreSQL based on context.

## Workflow

1.  **Analyze Requirements (Implicit & Explicit)**:
    - If the user provides a detailed requirement, follow it.
    - If the user provides ONLY a project name or a simple sentence (e.g., "Defect Management System"), you MUST **infer** the core business entities and functional modules based on general domain knowledge.
    - Example: For "Defect Management System", infer tables like `users`, `projects`, `defects` (or `issues`), `comments`, `attachments`.
2.  **Design Database**: Apply the Design Principles below.
3.  **Generate Output**: Produce the Markdown document and SQL.
4.  **Save to File**: Use the `Write` tool to save the complete output (Design Document + SQL) to a single Markdown file named `{project_name}_design.md` or `database_design.md`.

## Design Principles

1.  **Complexity & Scale**:
    - **Control Table Count**:
      - Small Projects: 4-6 tables.
      - Medium Projects: 6-10 tables.
      - **Limit**: Avoid exceeding 15 tables for personal-level projects.
    - **Avoid Over-Abstraction**: Do NOT create `base_entity` tables, multi-level inheritance tables, or complex generic relationship tables (EAV model).
    - **One Table = One Entity**: Each table must represent a clear business object (e.g., `users`, `projects`, `tasks`, `defects`, `orders`). Do NOT mix multiple concepts in one table.

2.  **Normalization**:
    - Follow **3rd Normal Form (3NF)**:
      - Each field must describe the current table's entity.
      - No redundant data storage.
      - Avoid repeating information.
    - Denormalize ONLY if there is a clear performance benefit and it's simple to maintain.

3.  **Naming Conventions**:
    - Tables: Plural, snake_case (e.g., `users`, `order_items`).
    - Columns: snake_case (e.g., `user_id`, `created_at`).
    - Primary Keys: `id` (BIGINT PRIMARY KEY AUTO_INCREMENT). **Avoid** composite keys or UUIDs unless strictly necessary.
    - Foreign Keys: `related_table_id` (e.g., `user_id`, `project_id`).

4.  **Field Types**:
    - **Status/Enums**: ALWAYS use Numeric types (`TINYINT` or `INT`) for status, priority, severity, type, etc. NEVER use ENUM or VARCHAR for these fields.
    - **Roles**: Use a dedicated `roles` table and a many-to-many relationship (`user_roles` table) if roles are dynamic or if users can have multiple roles. Even for simple roles, prefer numeric IDs over strings.
    - **No Field Abuse**: Do not use generic columns like `data1`, `data2` or `info` JSON columns unless the schema is truly dynamic.

5.  **Documentation (Comments)**:
    - **Crucial**: You MUST add SQL comments (`COMMENT '...'`) to every table and every column.
    - **Status Fields**: Explicitly explain numeric values (e.g., `COMMENT '0: Pending, 1: Active, 2: Closed'`).

6.  **Deletion Strategy**:
    - **Business Data** (e.g., Users, Orders, Posts): Use **Logical Deletion**. Add a column like `is_deleted` (BOOLEAN, default 0/FALSE) or `deleted_at` (TIMESTAMP, nullable).
    - **Temporary/Log Data** (e.g., Session tokens, Cache, Temporary uploads): Use **Physical Deletion**. No soft delete columns.

7.  **Standard Fields**:
    - All business tables MUST include `created_at` and `updated_at` timestamps.

8.  **Relationships**:
    - **One-to-One**: Foreign key on the "owner" side, or shared PK.
    - **One-to-Many**: Foreign key on the "many" side.
    - **Many-to-Many**: ALWAYS use an association/junction table (e.g., `user_roles`, `task_tags`).

9.  **Indexes**:
    - Add indexes ONLY for Foreign Keys and frequently queried fields (e.g., `email`, `username`, `status`). Do not over-index.

10. **Extensibility**:
    - Design should allow adding new features (e.g., new status types, new modules) without major schema refactoring.

## Output Format

You must output two distinct sections, and **save the entire content to a file**.

### 1. File Output Requirement

- **Action**: You MUST use the `Write` (or equivalent) tool to save the entire content (Design Document + SQL) to a file.
- **Filename**: Use `{project_name}_design.md` (snake_case). If no project name is clear, use `database_design.md`.

### 2. Database Design Document (Markdown)

- **Project Overview**: Brief summary.
- **Entity Relationships**: Explain the relationships between main entities.
- **Table Structures**:
  - Table Name & Description.
  - Columns: Name, Type, Constraints, **Comments/Description** (Explain what numeric values mean).
  - Deletion Strategy.

### 3. Executable SQL

- Provide a single code block with the complete SQL script.
- **Include Comments**: Use `COMMENT` syntax in `CREATE TABLE` statements for tables and columns.
- Ensure it runs without errors (e.g., correct ordering for foreign keys).
- Use the syntax appropriate for the requested database type.

## Example Interaction

**User**: "I need a database for a personal blog. Users can write posts and comment. Use MySQL."

**Response**:

### Database Design: Personal Blog

#### Tables

1.  **users**
    - Stores user account information.
    - Strategy: Logical Deletion (`deleted_at`).
    - Columns: `id`, `username`, `email`, `password_hash`, `created_at`, `updated_at`, `deleted_at`.

2.  **posts**
    - Blog entries.
    - Strategy: Logical Deletion (`deleted_at`).
    - Columns: `id`, `user_id` (FK), `title`, `content`, `published_at`, `created_at`, `updated_at`, `deleted_at`.

...

### SQL Script

```sql
-- Users Table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL, -- Logical deletion
    INDEX idx_email (email)
);

-- Posts Table
CREATE TABLE posts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    published_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL, -- Logical deletion
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```
