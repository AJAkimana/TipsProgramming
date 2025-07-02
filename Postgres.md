# Install postgres

`sudo apt update`
`sudo apt install postgresql postgresql-contrib`

## Configure Postgres
```
sudo -u postgres psql
postgres=# create database mydb;
postgres=# create user myuser with encrypted password 'mypass';
postgres=# grant all privileges on database mydb to myuser;
```

### change Password of a user
```
alter user <username> with encrypted password '<password>';
```

## Fix "ERROR: permission denied for schema public"

This error occurs when users try to create tables or perform operations in the public schema but lack the necessary permissions. This is a common issue, especially in newer PostgreSQL versions (15+) where the default permissions on the public schema have been changed for security reasons.

### Why this error occurs
- PostgreSQL 15+ removed the default CREATE permission for all users on the public schema
- The public schema permissions may have been explicitly revoked
- The user lacks USAGE and/or CREATE permissions on the public schema
- Database security policies restrict schema access

### Solution 1: Grant permissions to a specific user
Connect as a superuser (usually postgres) and grant the necessary permissions:

```sql
-- Connect as superuser
sudo -u postgres psql

-- Grant USAGE permission (allows user to access the schema)
GRANT USAGE ON SCHEMA public TO myuser;

-- Grant CREATE permission (allows user to create objects in the schema)
GRANT CREATE ON SCHEMA public TO myuser;

-- Grant all permissions on existing tables (if needed)
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO myuser;

-- Grant permissions on future tables (if needed)
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO myuser;
```

### Solution 2: Restore default public schema permissions
To restore the traditional behavior where all users can create in public schema:

```sql
-- Connect as superuser
sudo -u postgres psql

-- Grant CREATE and USAGE to all users (PUBLIC role)
GRANT CREATE, USAGE ON SCHEMA public TO PUBLIC;

-- Optionally, grant permissions on existing tables to all users
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO PUBLIC;
```

### Solution 3: Alternative approaches for different scenarios

#### For application-specific databases:
```sql
-- Create a dedicated schema for your application
CREATE SCHEMA IF NOT EXISTS myapp_schema;

-- Grant permissions to your application user
GRANT USAGE, CREATE ON SCHEMA myapp_schema TO myuser;

-- Set the search path so the user doesn't need to qualify table names
ALTER USER myuser SET search_path = myapp_schema, public;
```

#### For multiple users with different access levels:
```sql
-- Create a role for regular users
CREATE ROLE app_users;

-- Grant schema permissions to the role
GRANT USAGE, CREATE ON SCHEMA public TO app_users;

-- Add users to the role
GRANT app_users TO myuser;
GRANT app_users TO anotheruser;
```

#### To check current schema permissions:
```sql
-- Check schema permissions
\dn+

-- Check table permissions in public schema
\dp public.*

-- Check user's current permissions
\du myuser
```
