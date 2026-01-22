# Background

## Project Overview

IvorySQL is an Oracle-compatible PostgreSQL fork (based on PostgreSQL master) that maintains 100% compatibility with the latest PostgreSQL while adding Oracle-specific features. Key architectural components:

- **Dual Parser System**: Separate PostgreSQL and Oracle SQL parsers (`src/backend/parser/` and `src/backend/oracle_parser/`)
- **Compatibility Mode**: Runtime toggle between Oracle and PostgreSQL modes via `ivorysql.database_mode` GUC parameter
- **PL/iSQL**: Oracle PL/SQL-compatible procedural language (`src/pl/plisql/`)
- **Oracle Extension**: `contrib/ivorysql_ora/` provides Oracle data types, functions, and system views

# Build Instructions

Uses traditional Autoconf/Make with Meson as an alternative build system.

## Make Build Project Quick Start

### Primary Build: GNU Make (Autoconf)

1. **Clean build artifacts**
   ```bash
    make clean;
    make distclean;
    find . -name "*.o" -delete;
    git restore src/fe_utils/ora_string_utils.c
   ```

2. **Configure the build**

   **Configure with debug and assertion checks (recommended for development)**
   ```bash
   ./configure \
            --prefix=$PWD/inst_claude \
            --enable-cassert --enable-debug --enable-tap-tests --enable-rpath \
            --with-tcl --with-python --with-gssapi --with-pam --with-ldap \
            --with-openssl --with-libedit-preferred --with-uuid=e2fs \
            --with-ossp-uuid  --with-libxml --with-libxslt  --with-perl \
            --with-icu --with-libnuma --enable-injection-points
   ```

3. **Build the project**
   ```bash
    make -j$(nproc)
    cd contrib;
    make;
    cd -
   ```

4. **Install the build**
   ```bash
    make install;
    cd contrib;
    make install;
    cd -
   ```
### Alternative Build: Meson

1. **Clean the meson build and make build**
   # Requires Clean build artifacts

   ```bash
    rm -rf build;
    make clean;
    make distclean;
    find . -name "*.o" -delete;
    git restore src/fe_utils/ora_string_utils.c
   ```
2. **Configure the meson build**
   ```bash
    meson build -Dprefix=$PWD/inst_meson_claude
   ```
   When you encounter Conflicting files in source directory, remove the generated files as follows:
   ```
   rm -f  src/backend/oracle_parser/ora_kwlist_d.h 
   ```
3. **Build the project**
   ```bash
       ninja -C build
   ```

### Adding New Source Files

When adding a new `.c` file:
- Update `Makefile` - add to `OBJS` list (as `.o`)
- Update `meson.build` - add to `sources` list

## Testing

### Test Suites

IvorySQL has parallel test structures for PostgreSQL and Oracle compatibility:

- **PostgreSQL Tests**: `src/test/regress/`
- **Oracle Tests**: `src/oracle_test/regress/`

### Adding New Tests

When adding a new test:
- Update `Makefile` - add to `REGRESS` list
- Don't update `meson.build` - meson test not supported


### Running Tests

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

### Adding New Tests

**For Oracle packages (like DBMS_UTILITY):**
1. Create `contrib/ivorysql_ora/sql/<testname>.sql`
2. Create `contrib/ivorysql_ora/expected/<testname>.out`
3. Add `<testname>` to `ORA_REGRESS` in `contrib/ivorysql_ora/Makefile`
4. Run `make installcheck` to verify

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

## Directory Structure Overview

```
src/
├── backend/
│   ├── parser/           # PostgreSQL SQL parser
│   ├── oracle_parser/    # Oracle SQL parser
│   ├── catalog/          # System catalog tables
│   ├── commands/         # SQL command processing
│   ├── executor/         # Query execution engine
│   ├── optimizer/        # Query optimizer
│   └── tcop/             # Query dispatcher
├── pl/
│   ├── plisql/           # PL/iSQL (Oracle PL/SQL compatible)
│   ├── plpgsql/          # PostgreSQL PL/pgSQL
│   └── plperl/plpython/  # Other PL languages
├── interfaces/
│   ├── libpq/            # Client library
│   └── ecpg/             # Embedded SQL
├── bin/                  # Client utilities (psql, pg_dump, etc.)
├── test/                 # PostgreSQL regression tests
├── oracle_test/          # Oracle compatibility tests
└── include/              # Header files 
contrib/
├── ivorysql_ora/         # Oracle compatibility extension
├── ora_btree_gin/        # Oracle B-tree GIN index
└── ora_btree_gist/       # Oracle B-tree GIST index
```

### Working with Dual Parsers

The parser switching is automatic based on `ivorysql.database_mode`:
- `DB_PG` (0): Uses PostgreSQL parser (`src/backend/parser/`)
- `DB_ORACLE` (1): Uses Oracle parser (`src/backend/oracle_parser/`)

Key files:
- Oracle grammar: `src/backend/oracle_parser/ora_gram.y` 
- Oracle scanner: `src/backend/oracle_parser/ora_scan.l`
- Compatibility definitions: `src/include/utils/ora_compatible.h`

### PL/iSQL Development

PL/iSQL is in `src/pl/plisql/src/`:
- `pl_gram.y`: PL/SQL grammar 
- `pl_comp.c`: Compiler
- `pl_exec.c`: Executor 
- `pl_package.c`: Package support 
- Supports procedures, functions, packages, and autonomous transactions

### Oracle Extension

The `contrib/ivorysql_ora/` extension provides:
- Oracle data types (CHAR, VARCHAR2, DATE, TIMESTAMP, etc.)
- Oracle built-in functions
- Oracle system views (USER_TABLES, ALL_TABLES, etc.)
- MERGE statement
- DBMS_OUTPUT and DBMS_UTILITY packages
- Oracle XML functions
