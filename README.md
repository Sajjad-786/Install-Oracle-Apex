# Install Oracle APEX on Linux

A step-by-step guide to installing **Oracle Application Express (APEX)** on a Linux system using Oracle Database and ORDS (Oracle REST Data Services).

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Install Oracle Database](#2-install-oracle-database)
3. [Download and Install Oracle APEX](#3-download-and-install-oracle-apex)
4. [Configure APEX](#4-configure-apex)
5. [Install and Configure ORDS](#5-install-and-configure-ords)
6. [Set Up ORDS as a Service](#6-set-up-ords-as-a-service)
7. [Access Oracle APEX](#7-access-oracle-apex)
8. [Troubleshooting](#8-troubleshooting)

---

## 1. Prerequisites

### Hardware

| Resource | Minimum   | Recommended |
|----------|-----------|-------------|
| CPU      | 2 cores   | 4+ cores    |
| RAM      | 8 GB      | 16 GB       |
| Storage  | 50 GB     | 100+ GB     |

### Software

- **Operating System:** Oracle Linux 8/9, RHEL 8/9, or CentOS Stream 8/9
- **Oracle Database:** 19c, 21c, or 23c (Enterprise or Standard Edition)
- **Oracle APEX:** Version 24.x (latest version)
- **ORDS:** Version 24.x (must match the APEX version)
- **Java:** JDK 11 or higher

### Install Required Packages

```bash
sudo dnf install -y java-17-openjdk java-17-openjdk-devel wget unzip curl
```

Verify Java installation:

```bash
java -version
```

---

## 2. Install Oracle Database

> Skip this step if Oracle Database is already installed.

### 2.1 Set Up Oracle Linux Repository

```bash
sudo dnf install -y oracle-database-preinstall-19c
```

### 2.2 Download Oracle Database RPM

Download Oracle Database 19c from [oracle.com/downloads](https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html).

### 2.3 Install Oracle Database

```bash
sudo rpm -ivh oracle-database-ee-19c-1.0-1.x86_64.rpm
```

### 2.4 Create the Database

```bash
sudo /etc/init.d/oracledb_ORCLCDB-19c configure
```

### 2.5 Set Environment Variables

```bash
cat >> ~/.bash_profile << 'EOF'
export ORACLE_BASE=/opt/oracle
export ORACLE_HOME=/opt/oracle/product/19c/dbhome_1
export ORACLE_SID=ORCLCDB
export PATH=$PATH:$ORACLE_HOME/bin
EOF

source ~/.bash_profile
```

### 2.6 Test the Database Connection

```bash
sqlplus / as sysdba
```

```sql
SELECT status FROM v$instance;
-- Expected: OPEN
EXIT;
```

---

## 3. Download and Install Oracle APEX

### 3.1 Download APEX

```bash
cd /tmp
wget https://download.oracle.com/otn_software/apex/apex_24.1.zip
```

Alternatively, download manually from [apex.oracle.com/download](https://apex.oracle.com/download).

### 3.2 Extract APEX

```bash
unzip apex_24.1.zip -d /opt/oracle/
cd /opt/oracle/apex
```

### 3.3 Install APEX into the Database

Connect to the database as the `oracle` user via sqlplus:

```bash
sqlplus / as sysdba
```

```sql
-- Open the Pluggable Database (if using CDB)
ALTER SESSION SET CONTAINER = ORCLPDB1;

-- Install APEX
@apexins.sql SYSAUX SYSAUX TEMP /i/
```

> The installation takes approximately 15–30 minutes. Please wait.

### 3.4 Verify Installation Status

```sql
SELECT status FROM dba_registry WHERE comp_id = 'APEX';
-- Expected: VALID
```

---

## 4. Configure APEX

### 4.1 Set the APEX Admin Password

```bash
sqlplus / as sysdba
```

```sql
ALTER SESSION SET CONTAINER = ORCLPDB1;

BEGIN
    APEX_UTIL.set_security_group_id(10);
    APEX_UTIL.create_user(
        p_user_name       => 'ADMIN',
        p_email_address   => 'admin@example.com',
        p_web_password    => 'MySecurePassword123!',
        p_developer_privs => 'ADMIN'
    );
    COMMIT;
END;
/
```

Alternatively, use the APEX script:

```sql
@apxchpwd.sql
```

### 4.2 Enable APEX REST Configuration

```sql
ALTER SESSION SET CONTAINER = ORCLPDB1;
EXEC DBMS_XDB.sethttpport(0);
EXEC DBMS_XDB.setftpport(0);
COMMIT;
EXIT;
```

### 4.3 Deploy Static APEX Files

```bash
mkdir -p /var/www/html/i
cp -r /opt/oracle/apex/images/* /var/www/html/i/
```

---

## 5. Install and Configure ORDS

### 5.1 Download ORDS

```bash
cd /tmp
wget https://download.oracle.com/otn_software/java/ords/ords-latest.zip
unzip ords-latest.zip -d /opt/ords
```

Or install via Oracle Linux YUM repository:

```bash
sudo dnf install ords
```

### 5.2 Create the ORDS Configuration Directory

```bash
mkdir -p /etc/ords/config
```

### 5.3 Connect ORDS to the Database

```bash
/opt/ords/bin/ords --config /etc/ords/config install \
  --admin-user SYS \
  --db-hostname localhost \
  --db-port 1521 \
  --db-servicename ORCLPDB1 \
  --feature-db-api true \
  --feature-rest-enabled-sql true \
  --feature-sdw true \
  --log-folder /var/log/ords \
  --standalone-mode true \
  --standalone-http-port 8080 \
  --standalone-static-context-path /i \
  --standalone-static-path /var/www/html/i
```

> During installation, you will be prompted for the SYS password and an ORDS database user password.

### 5.4 Test ORDS

```bash
/opt/ords/bin/ords --config /etc/ords/config serve \
  --port 8080 \
  --secure false
```

Open in your browser: `http://localhost:8080/ords`

---

## 6. Set Up ORDS as a Service

### 6.1 Create a Systemd Service File

```bash
sudo tee /etc/systemd/system/ords.service << 'EOF'
[Unit]
Description=Oracle REST Data Services
After=network.target

[Service]
Type=simple
User=oracle
ExecStart=/opt/ords/bin/ords --config /etc/ords/config serve --port 8080
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF
```

### 6.2 Enable and Start the Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable ords
sudo systemctl start ords
sudo systemctl status ords
```

### 6.3 Open Firewall Port

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

---

## 7. Access Oracle APEX

After a successful installation, APEX is available at the following URLs:

| Page              | URL                                          |
|-------------------|----------------------------------------------|
| APEX Home         | `http://SERVER-IP:8080/ords/apex`            |
| APEX Admin        | `http://SERVER-IP:8080/ords/apex/apex_admin` |
| SQL Developer Web | `http://SERVER-IP:8080/ords/sql-developer`   |

**Default Credentials:**

- **Workspace:** `internal`
- **Username:** `ADMIN`
- **Password:** *(The password set in Step 4.1)*

---

## 8. Troubleshooting

### APEX Installation Fails

```sql
-- Check the error log
SELECT * FROM apex_install_progress ORDER BY install_time DESC;

-- Check APEX status
SELECT comp_id, comp_name, status, version FROM dba_registry WHERE comp_id = 'APEX';
```

### ORDS Does Not Start

```bash
# Check ORDS logs
sudo journalctl -u ords -f

# Validate the database connection
/opt/ords/bin/ords --config /etc/ords/config validate
```

### Port 8080 Not Reachable

```bash
# Check firewall status
sudo firewall-cmd --list-all

# Allow port in SELinux (if SELinux is active)
sudo semanage port -a -t http_port_t -p tcp 8080
```

### Reset Admin Password

```bash
sqlplus / as sysdba
```

```sql
ALTER SESSION SET CONTAINER = ORCLPDB1;
@/opt/oracle/apex/apxchpwd.sql
```

---

## Useful Links

- [Oracle APEX Official Documentation](https://docs.oracle.com/en/database/oracle/apex/)
- [ORDS Installation Guide](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/)
- [Oracle APEX Download](https://apex.oracle.com/download)
- [Oracle Database Download](https://www.oracle.com/database/technologies/)

---

## License

This project is licensed under the [MIT License](LICENSE).

---

*Created for the project: Install-Oracle-Apex*
