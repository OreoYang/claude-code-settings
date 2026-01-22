
IvorySQL regression testing guide covering PostgreSQL and Oracle compatibility test suites, including manual testing procedures for Oracle mode on port 1521.

## Regression Testing

### Regression Test Suites

IvorySQL has parallel test structures for PostgreSQL and Oracle compatibility:

- **PostgreSQL Tests**: `src/test/regress/`
- **Oracle Tests**: `src/oracle_test/regress/`

### Adding New Tests

When adding a new test:
- Update `Makefile` - add to `REGRESS` list
- Don't update `meson.build` - meson test not supported


### Running Regression Tests

**Note:**  Be sure you have already run make build the project before you do any testing.

1. **PostgreSQL Compatibility Tests** (`IvorySQL/`)
    - name: pg_regression
      run: make check-world

2. **Oracle Compatibility Tests** (`IvorySQL/`)
    - name: oracle_regression
      run: make oracle-check-world

3. **Oracle and pg regression Compatibility Tests** (`IvorySQL/`)
    - name: oracle_pg_regression
      run: make oracle-pg-check
    - name: pg_regression
      run: make check

4. **Oracle Extension Tests** (`contrib`)
    - name: contrib_regression
      run: |
        cd contrib && make oracle-check
        make check

### Test Pattern

All tests follow the same pattern:
1. SQL input files in `sql/` directory
2. Expected outputs in `expected/` directory (`.out` files)
3. Runner executes SQL and compares actual vs expected output

### Full CI Regression Tests (IMPORTANT)

Before submitting a PR, run the full regression test suites that CI runs:

**Expected output files that may need updating:**
- `src/oracle_test/regress/expected/*.out` - Oracle mode test expectations
- `src/test/regress/expected/*.out` - PostgreSQL mode test expectations
- `src/test/regress/expected/*_1.out` - Alternative expected outputs for different configurations

### Manual Testing with Oracle Compatibility

When you need to test SQL manually (e.g., testing PL/iSQL packages):

0. **Ensure backend progress postgres is stopped**
    ```bash
     pkill -9 postgres
    ```

1. **Initialize a test database in Oracle mode**
   ```bash
   cd inst_claude
   rm -rf test_oracle
   bin/initdb -D test_oracle -m oracle
   ```

2. **Start the database server**
   ```bash
   bin/pg_ctl -D test_oracle -l logfile start
   ```

   **Note:** Oracle mode servers listen on **both ports**:
   - Port 5432 (PostgreSQL default)
   - Port 1521 (Oracle default)

3. **Create a test database**
   ```bash
   bin/createdb testdb
   ```

4. **Connect and test (use Oracle port 1521)**
   ```bash
   # Interactive connection
   bin/psql -p 1521 -d testdb
   

   # Run a SQL file
   bin/psql -h localhost -p 1521 -d testdb -f /path/to/your/file.sql
   ```

5. **Stop the test server when done**
   ```bash
   bin/pg_ctl -D test_oracle stop -m fast
   ```

**Important:** Always use port **1521** when testing Oracle PL/SQL compatibility features (packages, PL/iSQL procedures, etc.)