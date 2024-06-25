# Cloud Database "Spawner"

With ElephantSQL going out of business, I need something new and most importantly, free.





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
