# Database Designer Lite Skill

A database design skill tailored for personal and small-to-medium scale projects. It generates clean, normalized, and extensible database schemas (Markdown documentation + Executable SQL) based on simple requirements, which are saved to a local `.md` file.

## Features

- **Smart Inference**: Automatically infers core entities and relationships from a project name or brief description.
- **Complexity Control**: Optimizes table count (4-15 tables) to avoid over-engineering while maintaining scalability.
- **Strict Standards**:
  - Enforces 3NF normalization.
  - Uses `BIGINT` auto-increment primary keys.
  - Requires numeric status fields (`TINYINT`) with SQL comments explaining values.
  - Prohibits generic EAV models or "god tables".
- **Documentation First**: Generates a Markdown design document with entity relationships and detailed table structures.
- **Production-Ready SQL**: Outputs executable SQL scripts with full comments (`COMMENT '...'`), indexes, and foreign key constraints.
- **Lifecycle Management**: Implements logical deletion for business data and physical deletion for temporary/log data.

## Installation

To use this skill, clone this repository and add it to your skill configuration (depending on your agent framework).

```bash
git clone https://github.com/your-username/database-designer-lite.git
```

## Usage

Trigger the skill by asking for a database design. You can provide a detailed requirement document or just a project name.

**Example Prompts:**

- "Design a database for a personal blog system using MySQL."
- "Create a database schema for a 'Defect Management System'. Use PostgreSQL."
- "I need a database for a Todo List app with categories and tags."

## Output Example

When you ask for a design, the skill provides:

1.  **Design Document**: A Markdown section explaining the project overview, relationships, and table definitions.
2.  **SQL Script**: A complete, commented SQL block ready to run.

```sql
-- Users Table
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY COMMENT 'Primary Key ID',
    username VARCHAR(50) NOT NULL UNIQUE COMMENT 'Login Username',
    password_hash VARCHAR(255) NOT NULL COMMENT 'Encrypted Password',
    is_active TINYINT DEFAULT 1 COMMENT 'Status: 0-Disabled, 1-Active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT 'Creation Time',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Last Update Time',
    deleted_at TIMESTAMP NULL COMMENT 'Logical Deletion Time'
) COMMENT='System User Information';
```

## License

MIT
