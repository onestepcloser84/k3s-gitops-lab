# PowerDNS Docker Compose Setup

This directory contains a complete Docker Compose setup for running PowerDNS Authoritative Server with MySQL backend, PowerDNS Admin web UI, and Adminer for database management.

## Overview

This setup provides:

- **PowerDNS Authoritative Server** (v4.8.4) - DNS server with MySQL backend
- **PowerDNS Admin** (v0.4.1) - Web-based management interface
- **MySQL 8.0** - Database backend for PowerDNS and PowerDNS Admin
- **Adminer** - Web-based database administration tool

## Prerequisites

- Docker and Docker Compose installed
- A local IP address to bind PowerDNS to (for DNS queries)
- Ports available:
  - `53/udp` and `53/tcp` (DNS)
  - `18081/tcp` (PowerDNS API)
  - `18082/tcp` (PowerDNS Admin UI)
  - `18080/tcp` (Adminer)

## Quick Start

### 1. Configure Environment Variables

Copy the example environment file and edit it:

```bash
cp env.example .env
```

Edit `.env` and set the following values:

```bash
# Bind PowerDNS only to the LAN IP
PDNS_BIND_IP=192.168.0.10  # Replace with your local IP address

# MySQL root password
MYSQL_ROOT_PASSWORD=your-secure-root-password

# MySQL password for normal user (used by PowerDNS)
DB_PASSWORD=your-secure-db-password

# PowerDNS API Key (use a long random string)
API_KEY=your-long-random-api-key-string
```

**Important**: Use strong, unique passwords and a secure API key. You can generate a random API key with:

```bash
openssl rand -hex 32
```

### 2. Configure Database Initialization

Copy the example SQL file and edit it:

```bash
cp mysql-init/create-dbs.sql.example mysql-init/create-dbs.sql
```

Edit `mysql-init/create-dbs.sql` and replace `'change-me-to-env-variable-DB_PASSWORD'` with the same password you set for `DB_PASSWORD` in your `.env` file:

```sql
CREATE USER IF NOT EXISTS 'pdns'@'%' IDENTIFIED BY 'your-secure-db-password';
-- ... rest of the file
```

**Note**: The password in `create-dbs.sql` must match the `DB_PASSWORD` value in your `.env` file.

### 3. Start the Services

Start all services:

```bash
docker-compose up -d
```

Check the status:

```bash
docker-compose ps
```

View logs:

```bash
# All services
docker-compose logs -f

### 4. Access the Services

- **PowerDNS API**: `http://<PDNS_BIND_IP>:18081/api/v1/servers/localhost`
  - Requires API key in header: `X-API-Key: <your-api-key>`
  
- **PowerDNS Admin UI**: `http://localhost:18082`
  - First-time setup: Follow the web interface to complete initial configuration
  
- **Adminer (Database UI)**: `http://localhost:18080`
  - System: `MySQL`
  - Server: `db`
  - Username: `root` or `pdns`
  - Password: Use `MYSQL_ROOT_PASSWORD` or `DB_PASSWORD` from `.env`
  - Database: `pdns` or `powerdnsadmin`

## Configuration Details

### PowerDNS Configuration

PowerDNS is configured via environment variables in `docker-compose.yml`:

- **Backend**: MySQL (`gmysql`)
- **API**: Enabled on port 8081
- **Webserver**: Enabled for API access
- **Mode**: Master (authoritative)
- **Log Level**: 4 (informational)

### Database Schema

The setup automatically initializes:

1. **MySQL databases**:
   - `pdns` - PowerDNS data (domains, records, etc.)
   - `powerdnsadmin` - PowerDNS Admin application data

2. **PowerDNS schema**: Automatically created in the `pdns` database

### Network Configuration

All services are connected to the `power-dns` Docker network, allowing them to communicate using service names (e.g., `db`, `powerdns`).

### Port Bindings

- **DNS (53/udp, 53/tcp)**: Bound to `${PDNS_BIND_IP}` - accessible from your network
- **PowerDNS API (18081/tcp)**: Bound to `${PDNS_BIND_IP}` - accessible from your network
- **PowerDNS Admin (18082/tcp)**: Bound to `127.0.0.1` - localhost only
- **Adminer (18080/tcp)**: Bound to `127.0.0.1` - localhost only

## Additional Resources

- [PowerDNS Documentation](https://doc.powerdns.com/)
- [PowerDNS Admin Documentation](https://github.com/PowerDNS-Admin/PowerDNS-Admin)
- [MySQL Documentation](https://dev.mysql.com/doc/)

