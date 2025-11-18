# Installing and Using Microsoft SQL Server on Ubuntu & Debian-based Linux

![image](./mssql-on-linux.jpg)

This guide provides a comprehensive, step-by-step walkthrough for installing and configuring Microsoft SQL Server on Debian-based Linux systems, including Ubuntu and Kali Linux. It covers the initial setup, installation of command-line tools, basic database operations, and common management tasks.

## Table of Contents

- [Part 1: Install SQL Server](#part-1-install-sql-server)
  - [Step 1: Import the Microsoft GPG Key](#step-1-import-the-microsoft-gpg-key)
  - [Step 2: Register the SQL Server Repository](#step-2-register-the-sql-server-repository)
  - [Step 3: Install SQL Server](#step-3-install-sql-server)
  - [Step 4: Configure SQL Server](#step-4-configure-sql-server)
  - [Step 5: Verify the Service and Open Firewall](#step-5-verify-the-service-and-open-firewall)
- [Part 2: Install SQL Server Command-Line Tools](#part-2-install-sql-server-command-line-tools)
  - [Step 1: Import GPG Key and Register Repository](#step-1-import-gpg-key-and-register-repository-for-tools)
  - [Step 2: Install mssql-tools](#step-2-install-mssql-tools)
  - [Step 3: Add Tools to Your PATH (Recommended)](#step-3-add-tools-to-your-path-recommended)
- [Part 3: Connect and Run Basic Queries](#part-3-connect-and-run-basic-queries)
  - [Step 1: Connect to SQL Server](#step-1-connect-to-sql-server)
  - [Step 2: Create a Database](#step-2-create-a-database)
  - [Step 3: Insert and Query Data](#step-3-insert-and-query-data)
  - [Step 4: Exit sqlcmd](#step-4-exit-sqlcmd)
- [Part 4: Common Management Tasks](#part-4-common-management-tasks)
  - [Resetting the 'sa' User Password](#resetting-the-sa-user-password)
  - [Re-running the Initial Configuration](#re-running-the-initial-configuration)

---

## Part 1: Install SQL Server

These steps will guide you through the installation of the main SQL Server package.

### Step 1: Import the Microsoft GPG Key

First, download the public GPG key and store it in the appropriate location for `apt`.

```bash
# Download the key, convert from ASCII to GPG format, and save it
curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | sudo gpg --dearmor -o /usr/share/keyrings/microsoft-prod.gpg
```



> ⚠️ **Note:** If you receive a warning about the public key not being available, you can use the following alternative command:
> ```bash
> curl https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc
> ```

### Step 2: Register the SQL Server Repository

Next, register the official Microsoft SQL Server repository for your Ubuntu version. The example below is for **Ubuntu 22.04**.

```bash
# Download the repository configuration
curl -fsSL https://packages.microsoft.com/config/ubuntu/22.04/mssql-server-preview.list | sudo tee /etc/apt/sources.list.d/mssql-server-preview.list
```

### Step 3: Install SQL Server

Update your package list and run the installation command.

```bash
sudo apt-get update
sudo apt-get install -y mssql-server
```

### Step 4: Configure SQL Server

After the installation is complete, run the setup script. This will prompt you to choose an edition and set the system administrator (`sa`) password.

```bash
sudo /opt/mssql/bin/mssql-conf setup
```

During the setup, you will be asked to:
1.  **Choose an edition:** Evaluation, Developer, and Express editions are freely licensed.
2.  **Set the `sa` password:** Your password must meet the SQL Server [password policy](https://learn.microsoft.com/en-us/sql/relational-databases/security/password-policy?view=sql-server-ver17). It must be at least 8 characters long and contain characters from three of the following four categories: uppercase letters, lowercase letters, digits, and symbols.

### Step 5: Verify the Service and Open Firewall

Once the configuration is done, check that the SQL Server service is running.

```bash
systemctl status mssql-server --no-pager
```

If you plan to connect remotely, remember to open the default SQL Server TCP port **1433** on your firewall.

---

## Part 2: Install SQL Server Command-Line Tools

To interact with your new SQL Server instance, you need tools like `sqlcmd` and `bcp`.

### Step 1: Import GPG Key and Register Repository for Tools

This repository is for the client tools and may be different from the server repository.

```bash
# Import the public GPG key (may be the same as before)
curl https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc

# Register the Microsoft Ubuntu repository for the tools
# This example uses the 18.04 repo, which is widely compatible
curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list | sudo tee /etc/apt/sources.list.d/mssql-release.list
```

### Step 2: Install mssql-tools

Update the package list and install the tools package along with the `unixODBC` developer package, which is a dependency.

```bash
sudo apt-get update
sudo apt-get install -y mssql-tools18 unixodbc-dev
```

### Step 3: Add Tools to Your PATH (Recommended)

To use `sqlcmd` and `bcp` without typing the full path, add their directory to your `PATH` environment variable.

- **For login sessions** (when you first log in to the machine):

```bash
echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' >> ~/.bash_profile
source ~/.bash_profile
```

- **For interactive/non-login sessions** (new terminal windows):

```bash
echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' >> ~/.bashrc
source ~/.bashrc
```

---

## Part 3: Connect and Run Basic Queries

Now you can connect to your local SQL Server instance and perform basic database operations.

### Step 1: Connect to SQL Server

Use `sqlcmd` with the server name (`-S`), username (`-U`), and password (`-P`).

> 💡 **Tip:** Replace `<password>` with the `sa` password you created during setup.

```bash
sqlcmd -S localhost -U sa -P '<password>'
```

If successful, you will see a `1>` prompt.

### Step 2: Create a Database

At the `sqlcmd` prompt, enter the following Transact-SQL (T-SQL) commands to create a database and verify its creation.

```sql
CREATE DATABASE TestDB;
SELECT Name FROM sys.databases;
GO
```

### Step 3: Insert and Query Data

Let's create a table, insert some data, and then query it.

```sql
-- Switch to the context of your new database
USE TestDB;
GO

-- Create a new table
CREATE TABLE dbo.Inventory (
    id INT,
    name NVARCHAR(50),
    quantity INT,
    PRIMARY KEY (id)
);
GO

-- Insert two rows of data
INSERT INTO dbo.Inventory VALUES (1, 'banana', 150);
INSERT INTO dbo.Inventory VALUES (2, 'orange', 154);
GO

-- Select data from the table
SELECT * FROM dbo.Inventory WHERE quantity > 152;
GO
```

### Step 4: Exit sqlcmd

To end your session, type `QUIT`.

```sql
QUIT
```

---

## Part 4: Common Management Tasks

### Resetting the 'sa' User Password

If you forget the `sa` password, you can reset it using one of the following methods.

#### Method 1: Using `mssql-conf` (Recommended)

This is the standard and safest way to reset the password.

```bash
# 1. Stop the SQL Server service
sudo systemctl stop mssql-server

# 2. Run the set-sa-password command
sudo /opt/mssql/bin/mssql-conf set-sa-password
```
You will be prompted to enter a new password. After setting it, restart the service with `sudo systemctl start mssql-server`.

#### Method 2: Manual Process (If Method 1 fails)

This method involves manually stopping the process and then running the password reset command. **Use with caution.**

```bash
# 1. Switch to the root user
su

# 2. Find and forcefully kill the sqlservr process
PID=$(pgrep sqlservr)
if [ -n "$PID" ]; then
    echo "Killing SQL Server process with PID: $PID"
    kill -9 $PID
else
    echo "SQL Server process not found."
fi

# 3. Verify that the service is stopped
systemctl status mssql-server --no-pager

# 4. Run the password reset command
/opt/mssql/bin/mssql-conf set-sa-password

# 5. After setting the password, exit the root shell and start the service
exit
sudo systemctl start mssql-server
```

### Re-running the Initial Configuration

If you need to change the SQL Server edition or perform a full reconfiguration, you can run the `setup` command again.

> ⚠️ **Warning:** This may reset existing configurations.

```bash
sudo /opt/mssql/bin/mssql-conf setup
```
```





