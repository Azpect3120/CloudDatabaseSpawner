# Cloud Database "Spawner"

With ElephantSQL going out of business, I need something new and most importantly, free.


## My Take

Need to make a user in PostgreSQL with a defined password and then create a database with the same name as the username.

Create a user:
```bash
# postgres should be replaced with the psql superuser who's password you have
# I am not sure how this will work without a password

sudo -i -u postgres psql -c "CREATE USER username WITH PASSWORD 'password';"
```

```sql
CREATE USER username WITH PASSWORD 'password';
```

Create the database:
```bash
sudo -i -u postgres psql -c "CREATE DATABASE username;"
sudo -i -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE username TO username;"
```

```sql
CREATE DATABASE username;
GRANT ALL PRIVILEGES ON DATABASE username TO username;
```

Connect to the database:
```bash
psql -h your_server_ip -U username -d username
```

### Found this fix: [stackoverflow.com](https://stackoverflow.com/questions/67276391/why-am-i-getting-a-permission-denied-error-for-schema-public-on-pgadmin-4)

```bash
CREATE DATABASE EXAMPLE_DB;
CREATE USER EXAMPLE_USER WITH ENCRYPTED PASSWORD 'Sup3rS3cret';
GRANT ALL PRIVILEGES ON DATABASE EXAMPLE_DB TO EXAMPLE_USER;
\c EXAMPLE_DB postgres
# You are now connected to database "EXAMPLE_DB" as user "postgres".
GRANT ALL ON SCHEMA public TO EXAMPLE_USER;
```

### Help from ChatGPT
```bash
#!/bin/bash

# Define variables
DB_NAME="EXAMPLE_DB"
DB_USER="EXAMPLE_USER"
DB_PASS="Sup3rS3cret"

# Create database
psql -U postgres -c "CREATE DATABASE $DB_NAME;"

# Create user with password
psql -U postgres -c "CREATE USER $DB_USER WITH ENCRYPTED PASSWORD '$DB_PASS';"

# Grant privileges on the database to the user
psql -U postgres -c "GRANT ALL PRIVILEGES ON DATABASE $DB_NAME TO $DB_USER;"

# Connect to the database and grant privileges on public schema
psql -U postgres -d $DB_NAME -c "GRANT ALL ON SCHEMA public TO $DB_USER;"
```


## I asked ChatGPT
Building a "cloud database" on your Ubuntu server involves several key steps: setting up PostgreSQL, managing user accounts, and ensuring secure access to individual databases. Here’s a detailed guide to help you through the process:

### Step 1: Install PostgreSQL

1. **Update your package list:**
   ```bash
   sudo apt update
   ```

2. **Install PostgreSQL:**
   ```bash
   sudo apt install postgresql postgresql-contrib
   ```

3. **Start and enable PostgreSQL:**
   ```bash
   sudo systemctl start postgresql
   sudo systemctl enable postgresql
   ```

### Step 2: Secure PostgreSQL

1. **Switch to the PostgreSQL user:**
   ```bash
   sudo -i -u postgres
   ```

2. **Access the PostgreSQL prompt:**
   ```bash
   psql
   ```

3. **Set a password for the `postgres` user:**
   ```sql
   \password postgres
   ```

4. **Exit the prompt:**
   ```sql
   \q
   ```

### Step 3: Create User Management Script

You can create a bash script to automate the creation of users and databases.

1. **Create a script file:**
   ```bash
   sudo nano /usr/local/bin/create_pg_user_db.sh
   ```

2. **Add the following script to create a PostgreSQL user and database:**
   ```bash
   #!/bin/bash

   if [ $# -ne 2 ]; then
       echo "Usage: $0 username password"
       exit 1
   fi

   USERNAME=$1
   PASSWORD=$2

   # Create a PostgreSQL user
   sudo -i -u postgres psql -c "CREATE USER $USERNAME WITH PASSWORD '$PASSWORD';"
   # Create a database with the same name as the user and grant all privileges to the user
   sudo -i -u postgres psql -c "CREATE DATABASE $USERNAME;"
   sudo -i -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE $USERNAME TO $USERNAME;"

   echo "User $USERNAME and database $USERNAME created successfully."
   ```

3. **Make the script executable:**
   ```bash
   sudo chmod +x /usr/local/bin/create_pg_user_db.sh
   ```

### Step 4: Create Users and Databases

Use the script to create new users and their corresponding databases.

1. **Run the script:**
   ```bash
   sudo /usr/local/bin/create_pg_user_db.sh newuser newpassword
   ```

### Step 5: Configure PostgreSQL for Remote Access

1. **Edit the `postgresql.conf` file to allow remote connections:**
   ```bash
   sudo nano /etc/postgresql/12/main/postgresql.conf
   ```
   Change the line:
   ```plaintext
   #listen_addresses = 'localhost'
   ```
   to:
   ```plaintext
   listen_addresses = '*'
   ```

2. **Edit the `pg_hba.conf` file to allow remote connections:**
   ```bash
   sudo nano /etc/postgresql/12/main/pg_hba.conf
   ```
   Add the following line to allow connections from any IP (replace `12` with your PostgreSQL version):
   ```plaintext
   host    all             all             0.0.0.0/0               md5
   ```

3. **Restart PostgreSQL to apply changes:**
   ```bash
   sudo systemctl restart postgresql
   ```

### Step 6: Ensure Security and Firewall Configuration

1. **Configure UFW to allow connections to PostgreSQL:**
   ```bash
   sudo ufw allow 5432/tcp
   ```

2. **Enable UFW if it’s not already enabled:**
   ```bash
   sudo ufw enable
   ```

3. **Consider using SSL/TLS for encrypted connections** by generating certificates and configuring PostgreSQL to use them.

### Step 7: Provide Users with Connection Information

Give your users the connection details, including:

- **Hostname/IP** of your server.
- **Port number** (default is 5432).
- **Database name** (same as their username).
- **Username** and **password**.

Example connection string:
```plaintext
psql -h your_server_ip -U username -d username
```

### Additional Considerations

1. **Backup and Recovery:**
   Implement a backup strategy using tools like `pg_dump` or `pg_basebackup` to ensure data can be recovered in case of failure.

2. **Monitoring and Performance:**
   Use monitoring tools like `pgAdmin`, `Prometheus`, or `Grafana` to keep an eye on the performance and health of your PostgreSQL server.

3. **Scalability:**
   If you expect a high number of users or databases, consider setting up connection pooling using tools like `pgbouncer` to manage resources efficiently.

4. **Automation:**
   For large-scale user management, consider integrating your script with a web interface or an API to automate user creation.

Following these steps will help you set up a functional cloud database on your Ubuntu server, providing users with unique access to their PostgreSQL databases.



Creating a user in PostgreSQL does not require creating a corresponding Linux system user unless your setup specifically demands it for some reason. For most use cases, managing users directly within PostgreSQL is sufficient. Here's a detailed explanation and the rationale behind this approach.

### Why Creating Linux System Users is Not Necessary

1. **Database Isolation:** 
   - PostgreSQL users and roles are entirely managed within the PostgreSQL system, independent of the underlying operating system users. This allows for easier management and better isolation of database users.
   - Users authenticate and connect to the PostgreSQL server without needing corresponding system user accounts.

2. **Security and Simplicity:**
   - Managing users within PostgreSQL provides a cleaner and more secure setup by avoiding the need to grant additional permissions at the OS level.
   - It simplifies the process, reducing potential security risks associated with unnecessary system accounts.

3. **Resource Management:**
   - PostgreSQL can handle user permissions, roles, and resource quotas without requiring OS-level management.
   - This is especially useful when scaling up, as managing numerous system users can become cumbersome and error-prone.

4. **Access Control:**
   - PostgreSQL provides robust mechanisms to control access and permissions at the database level, making system users unnecessary for database access control.

### When You Might Consider Creating Linux System Users

There are specific scenarios where creating Linux system users could be necessary or beneficial:

1. **File-Level Access:**
   - If you need users to have access to the filesystem for tasks such as file uploads or backups, you might create corresponding Linux users.
   - In this case, you’ll need to manage both PostgreSQL and system users and ensure proper synchronization between them.

2. **Enhanced Security Policies:**
   - In some high-security environments, you may want to enforce additional controls at the OS level, such as integrating with LDAP or other centralized authentication mechanisms.
   - This allows for more granular control over user activities and resource access.

### Practical Steps for PostgreSQL User Management

For a typical cloud database setup, managing users within PostgreSQL itself is usually sufficient. Here’s how you can manage users and their databases:

#### Step 1: Create a PostgreSQL User and Database

1. **Log in to PostgreSQL:**
   ```bash
   sudo -i -u postgres psql
   ```

2. **Create a PostgreSQL user and database:**
   ```sql
   CREATE USER newuser WITH PASSWORD 'newpassword';
   CREATE DATABASE newuserdb OWNER newuser;
   ```

3. **Grant necessary privileges:**
   ```sql
   GRANT ALL PRIVILEGES ON DATABASE newuserdb TO newuser;
   ```

4. **Exit psql:**
   ```sql
   \q
   ```

#### Step 2: Configure Remote Access

1. **Edit PostgreSQL configuration:**
   - Modify `postgresql.conf` and `pg_hba.conf` as detailed in the previous steps to allow remote connections and secure access.

2. **Restart PostgreSQL:**
   ```bash
   sudo systemctl restart postgresql
   ```

#### Step 3: Test the Connection

Provide users with the connection details, and ensure they can connect to their database remotely using their PostgreSQL credentials.

#### Example Connection Command:
```bash
psql -h your_server_ip -U newuser -d newuserdb
```

### Conclusion

For most use cases involving a PostgreSQL database on an Ubuntu server, you do not need to create Linux system users. PostgreSQL’s internal user management provides all the necessary tools to handle authentication, access control, and database management effectively and securely.

If you do have specific requirements that involve system-level access or enhanced security policies, then creating Linux users in addition to PostgreSQL users might be necessary. However, these scenarios are less common and typically involve more complex setups.


Yes, you can run `psql -c` commands without requiring a password by leveraging several methods. Each method involves configuring PostgreSQL and/or your system to allow password-less access for the specific scenarios you need. Here’s how you can achieve this:

### Method 1: Use `.pgpass` File

The `.pgpass` file allows you to store your PostgreSQL credentials in a file so that `psql` can access them without prompting for a password.

#### Steps to Create a `.pgpass` File

1. **Create/Edit the `.pgpass` file:**
   ```bash
   nano ~/.pgpass
   ```

2. **Add the credentials:**
   ```plaintext
   hostname:port:database:username:password
   ```
   - For example:
     ```plaintext
     localhost:5432:postgres:postgres:your_password
     ```

3. **Set the file permissions to restrict access:**
   ```bash
   chmod 600 ~/.pgpass
   ```

4. **Run the `psql` command:**
   ```bash
   psql -h localhost -U postgres -d your_database -c "SELECT version();"
   ```
   - This will not prompt for a password if the `.pgpass` file contains the correct credentials.

### Method 2: Use Peer Authentication

Peer authentication uses the operating system user to authenticate the PostgreSQL user without requiring a password.

#### Steps to Configure Peer Authentication

1. **Edit the `pg_hba.conf` file:**
   ```bash
   sudo nano /etc/postgresql/12/main/pg_hba.conf
   ```
   - Note: Replace `12` with your PostgreSQL version.

2. **Set `peer` authentication for local connections:**
   ```plaintext
   local   all             all                                     peer
   ```

3. **Restart PostgreSQL:**
   ```bash
   sudo systemctl restart postgresql
   ```

4. **Run the `psql` command:**
   ```bash
   sudo -u postgres psql -c "SELECT version();"
   ```
   - Since you are logged in as the `postgres` system user, you don’t need to provide a password.

### Method 3: Use Environment Variables

You can set environment variables for PostgreSQL connection parameters, including the password.

#### Steps to Set Environment Variables

1. **Set the `PGPASSWORD` environment variable:**
   ```bash
   export PGPASSWORD='your_password'
   ```

2. **Run the `psql` command:**
   ```bash
   psql -U postgres -h localhost -d your_database -c "SELECT version();"
   ```

3. **Unset the `PGPASSWORD` variable after use:**
   ```bash
   unset PGPASSWORD
   ```

### Method 4: Use a Connection Service File

A PostgreSQL service file allows you to define connection parameters in a file and reference them without needing to supply a password manually.

#### Steps to Create a Connection Service File

1. **Create the `pg_service.conf` file:**
   ```bash
   nano ~/.pg_service.conf
   ```

2. **Add service definitions:**
   ```plaintext
   [mydb]
   host=localhost
   port=5432
   dbname=your_database
   user=postgres
   password=your_password
   ```

3. **Set the `PGSERVICE` environment variable:**
   ```bash
   export PGSERVICE=mydb
   ```

4. **Run the `psql` command:**
   ```bash
   psql -c "SELECT version();"
   ```

### Method 5: Use Trust Authentication

`Trust` authentication allows connections without requiring a password, but it should be used carefully due to security risks.

#### Steps to Configure Trust Authentication

1. **Edit the `pg_hba.conf` file:**
   ```bash
   sudo nano /etc/postgresql/12/main/pg_hba.conf
   ```

2. **Set `trust` authentication for local connections:**
   ```plaintext
   local   all             all                                     trust
   ```

3. **Restart PostgreSQL:**
   ```bash
   sudo systemctl restart postgresql
   ```

4. **Run the `psql` command:**
   ```bash
   psql -U postgres -h localhost -d your_database -c "SELECT version();"
   ```

- Note: Trust authentication means any local user can connect to the database without a password, which poses a security risk.

### Security Considerations

- **Environment Variables and `.pgpass`**: Be cautious with `.pgpass` files and environment variables. They should be protected with appropriate permissions to prevent unauthorized access.
- **Trust and Peer Authentication**: Use these methods only in secure and controlled environments where you trust all local users, as they bypass password requirements completely.

By following these methods, you can run `psql -c` commands without requiring a password, facilitating automated scripts and operations without manual password entry.
