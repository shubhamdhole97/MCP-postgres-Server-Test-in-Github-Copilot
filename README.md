# PostgreSQL MCP Server Setup for GitHub Copilot

This guide explains how to install PostgreSQL, create a sample database, configure the MCP database server, and use it with GitHub Copilot.

---

## 1. Install Node.js and npm

Check if Node.js and npm are installed:

```bash
node -v
npm -v
```

If missing, install them:

```bash
sudo apt update
sudo apt install nodejs npm -y
```

---

## 2. Install PostgreSQL

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```

Check PostgreSQL status:

```bash
sudo systemctl status postgresql
```

Start and enable PostgreSQL if needed:

```bash
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

Check version:

```bash
psql --version
```

---

## 3. Create PostgreSQL Database and User

Login as postgres user:

```bash
sudo -i -u postgres
psql
```

Run:

```sql
CREATE DATABASE company_db;

CREATE USER user1 WITH PASSWORD '123';

GRANT ALL PRIVILEGES ON DATABASE company_db TO user1;

\q
```

Exit postgres Linux user:

```bash
exit
```

---

## 4. Fix Schema Permission

If you get this error:

```text
ERROR: permission denied for schema public
```

Run:

```bash
sudo -u postgres psql -d company_db
```

Inside psql:

```sql
GRANT ALL ON SCHEMA public TO user1;
ALTER SCHEMA public OWNER TO user1;
GRANT ALL PRIVILEGES ON DATABASE company_db TO user1;
\q
```

---

## 5. Test PostgreSQL Connection

```bash
psql -h 127.0.0.1 -U user1 -d company_db
```

Password:

```text
123
```

Expected prompt:

```text
company_db=>
```

Exit:

```sql
\q
```

---

## 6. Create Sample Tables and Data

Connect:

```bash
psql -h 127.0.0.1 -U user1 -d company_db
```

Run:

```sql
CREATE TABLE departments (
    department_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    employee_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    employee_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    department_id INT,
    CONSTRAINT fk_department
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);

CREATE TABLE projects (
    project_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    project_name VARCHAR(100) NOT NULL,
    employee_id INT,
    CONSTRAINT fk_employee
        FOREIGN KEY (employee_id)
        REFERENCES employees(employee_id)
);

INSERT INTO departments (department_name) VALUES
('HR'),
('Finance'),
('IT'),
('Sales'),
('Marketing'),
('Operations'),
('Support'),
('Legal'),
('Research'),
('Administration');

INSERT INTO employees (employee_name, email, department_id) VALUES
('John Smith', 'john@example.com', 1),
('Alice Brown', 'alice@example.com', 2),
('Bob Wilson', 'bob@example.com', 3),
('Emma Davis', 'emma@example.com', 4),
('David Lee', 'david@example.com', 5),
('Sophia Clark', 'sophia@example.com', 6),
('Michael Hall', 'michael@example.com', 7),
('Olivia Young', 'olivia@example.com', 8),
('James King', 'james@example.com', 9),
('Charlotte Scott', 'charlotte@example.com', 10),
('Shubham Dhole', 'shubhamdhole98@gmail.com', 3);

INSERT INTO projects (project_name, employee_id) VALUES
('Payroll System', 1),
('Budget Analysis', 2),
('Cloud Migration', 3),
('CRM Upgrade', 4),
('SEO Campaign', 5),
('Warehouse Automation', 6),
('Customer Portal', 7),
('Contract Review', 8),
('AI Research', 9),
('Office Renovation', 10),
('GitHub MCP Testing', 11);
```

---

## 7. Verify Data

```sql
SELECT * FROM departments;

SELECT * FROM employees;

SELECT * FROM projects;
```

Join query:

```sql
SELECT
    d.department_name,
    e.employee_name,
    e.email,
    p.project_name
FROM departments d
JOIN employees e
    ON d.department_id = e.department_id
JOIN projects p
    ON e.employee_id = p.employee_id;

```

Exit:

```sql
\q
```

---

## 8. Install MCP Database Server Globally

```bash
sudo npm install -g @executeautomation/database-server
```

Confirm CLI is installed:

```bash
ea-database-server --help
```

If command is not found:

```bash
export PATH=$PATH:$(npm config get prefix)/bin
```

---

## 9. Run PostgreSQL MCP Server Manually

```bash
ea-database-server \
  --postgres \
  --host 127.0.0.1 \
  --database company_db \
  --port 5432 \
  --user user1 \
  --password 123
```

Expected output:

```text
[INFO] Initializing postgresql database...
[INFO] PostgreSQL connection established successfully
[INFO] Connected to PostgreSQL database
[INFO] Starting MCP server...
[INFO] Server running. Press Ctrl+C to exit.
```

Keep this terminal running.

---

## 10. GitHub Copilot MCP Configuration

Add this MCP config:

```json
{
    "mcpServers": {
        "postgres": {
            "command": "ea-database-server",
            "args": [
                "--postgres",
                "--host",
                "127.0.0.1",
                "--database",
                "company_db",
                "--port",
                "5432",
                "--user",
                "user1",
                "--password",
                "123"
            ]
        }
    }
}
```

Restart VS Code or GitHub Copilot after adding the config.

---

## 11. Alternative MCP Method Using npx

If direct command does not work, use npx:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "@executeautomation/database-server",
        "--postgres",
        "--host", "127.0.0.1",
        "--database", "company_db",
        "--port", "5432",
        "--user", "user1",
        "--password", "123"
      ]
    }
  }
}
```

---

## 12. Test in GitHub Copilot Chat

Ask Copilot:

```text
Show all employees from PostgreSQL database.
```

```text
Find employee Shubham Dhole.
```

```text
Show department, employee name, email, and project name.
```

---

## 13. Debug Commands

Check installed package:

```bash
npm list -g --depth=0
```

Find binary:

```bash
ls $(npm config get prefix)/bin
```

Check npm prefix:

```bash
npm config get prefix
```

Test CLI:

```bash
ea-database-server --help
```

Check PostgreSQL port:

```bash
sudo ss -tulnp | grep 5432
```

Check PostgreSQL users:

```bash
sudo -u postgres psql -c "\du"
```

Check databases:

```bash
sudo -u postgres psql -c "\l"
```

---

## Final Working Command

```bash
ea-database-server --postgres --host 127.0.0.1 --database company_db --port 5432 --user user1 --password 123
```
