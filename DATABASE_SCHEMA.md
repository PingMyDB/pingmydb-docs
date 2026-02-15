# Database Schema

PingMyDb uses PostgreSQL to store relational data and monitor logs.

## Core Tables

### `users`
| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | UUID/Serial | Primary key |
| `name` | VARCHAR | User's full name |
| `email` | VARCHAR | Unique login email |
| `password` | VARCHAR | Hashed password |
| `plan` | VARCHAR | free, student, pro, enterprise |
| `student_status` | VARCHAR | none, pending, verified |
| `institution_name` | VARCHAR | (Optional) For students |
| `created_at` | TIMESTAMP | Signup time |

### `monitors`
| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | Serial | Primary key |
| `user_id` | UUID | Foreign key to `users` |
| `name` | VARCHAR | Friendly monitor name |
| `type` | VARCHAR | postgres, mysql, mongodb |
| `db_uri` | TEXT | **Encrypted** connection string |
| `interval_ms` | INTEGER | Ping frequency |
| `status` | VARCHAR | online, offline, paused |
| `is_paused` | BOOLEAN | Operational toggle |

### `monitor_checks`
| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | Serial | Primary key |
| `monitor_id` | INTEGER | Foreign key to `monitors` |
| `latency_ms` | INTEGER | Round-trip time |
| `status` | VARCHAR | UP, DOWN |
| `error_msg` | TEXT | (Optional) Failure details |
| `checked_at` | TIMESTAMP | Execution timestamp |

### `notification_channels`
| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | Serial | Primary key |
| `user_id` | UUID | Foreign key to `users` |
| `type` | VARCHAR | email, slack, discord |
| `config` | JSONB | Webhook URLs or target emails |
| `is_enabled` | BOOLEAN | Toggle status |
