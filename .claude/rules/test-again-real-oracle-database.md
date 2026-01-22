## Testing Against Real Oracle Database

A real Oracle Database Free container is available for validating Oracle compatibility.

### Container Information

- **Container name:** `ivorysql-oracle-1`
- **Image:** `container-registry.oracle.com/database/free:23.26.0.0-lite`
- **Version:** Oracle 23.26 Free
- **Status:** Optional; requires `--profile ora` flag to start
- **Memory:** Requires minimum 3GB RAM

**IMPORTANT - Check before starting:**
The Oracle container requires 3GB+ RAM. Multiple git worktrees can share one instance.
**ALWAYS** check for existing Oracle containers before starting a new one:
```bash
docker ps --filter "ancestor=container-registry.oracle.com/database/free:23.26.0.0-lite"
```
If an Oracle container is already running, use `docker exec` to connect to it directly. Do NOT start another instance.

**Starting the Oracle container (only if none exists):**
```bash
docker compose --profile ora up -d
```

### Connecting to Oracle

**Interactive SQL*Plus session:**
```bash
docker exec -it ivorysql-oracle-1 sqlplus / as sysdba
```

**Run SQL from command line:**
```bash
docker exec ivorysql-oracle-1 bash -c "echo 'SELECT * FROM dual;' | sqlplus -s / as sysdba"
```

**Run inline SQL script:**
```bash
docker exec ivorysql-oracle-1 bash -c "cat << 'EOF' | sqlplus -s / as sysdba
SET SERVEROUTPUT ON;
DECLARE
  result NUMBER;
BEGIN
  SELECT (100 - 50) * 0.01 INTO result FROM dual;
  DBMS_OUTPUT.PUT_LINE('Result: ' || result);
END;
/
EXIT;
EOF
"
```