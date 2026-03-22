# Fix Subscription Manager Build Errors

## TL;DR

> **Quick Summary**: Fix 3 critical blockers preventing the Spring Boot application from compiling and starting.
> 
> **Deliverables**: 
> - Fixed `pom.xml` with correct Java version
> - Created Flyway migration file
> - Fixed misplaced `package-info.java`
> - Updated `application.yml` with Redis and Flyway configuration
> 
> **Estimated Effort**: Quick (15-20 minutes)
> **Parallel Execution**: YES - all fixes are independent
> **Critical Path**: pom.xml fix → compile → app start verification

---

## Context

### Original Problem
User reported "code base is not working" - after thorough analysis, found 3 critical blockers and 2 configuration issues.

### Research Findings
- **Java 17** is installed on system (`/usr/lib/jvm/java-17-openjdk-amd64`)
- **pom.xml** specifies Java 21, causing: `release version 21 not supported`
- **No Flyway migrations** exist at `src/main/resources/db/migration/`
- **package-info.java** has wrong package declaration: `subscription_manager.src.main.java`
- PostgreSQL configured but Redis starter included without Redis config

---

## Work Objectives

### Core Objective
Fix all blocking build errors so the Spring Boot application compiles and starts successfully.

### Concrete Deliverables
- `pom.xml` - Java 17 (compatible with system)
- `src/main/resources/db/migration/V1__initial_schema.sql` - Flyway migration
- `src/main/java/package-info.java` - Fixed or deleted
- `application.yml` - Redis configuration added
- Verified compilation with `./mvnw compile`
- Verified startup with `./mvnw spring-boot:run`

### Definition of Done
- [ ] `./mvnw compile` succeeds with 0 errors
- [ ] Application starts without Flyway migration errors
- [ ] All 3 critical blockers resolved

### Must Have
- Application compiles without errors
- Flyway can find migrations
- Package structure is valid

### Must NOT Have
- No changes to business logic
- No new features
- No test modifications

---

## Verification Strategy

### Test Decision
- **Infrastructure exists**: NO (minimal fixes, no tests needed for infrastructure)
- **Automated tests**: None
- **Framework**: None

### QA Policy
Verification is manual via Maven commands.

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (All fixes can be done in parallel):
├── Task 1: Fix pom.xml Java version
├── Task 2: Create Flyway migration directory and file
├── Task 3: Fix or delete misplaced package-info.java
└── Task 4: Add Redis configuration to application.yml

Wave 2 (Sequential - must wait for fixes):
├── Task 5: Verify compilation
└── Task 6: Verify application startup
```

---

## TODOs

- [ ] 1. Fix pom.xml Java version to 17

  **What to do**:
  - Change `<java.version>21</java.version>` to `<java.version>17</java.version>`
  - Also change maven-compiler-plugin release if present

  **Must NOT do**:
  - Don't change any other dependencies

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Simple XML property change, single file
  - **Skills**: None required

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 2, 3, 4)
  - **Blocks**: Task 5 (compile verification)
  - **Blocked By**: None

  **References**:
  - `pom.xml:30` - Current Java version setting

  **Acceptance Criteria**:
  - [ ] pom.xml contains `<java.version>17</java.version>`
  - [ ] No other pom.xml changes made

  **QA Scenarios**:

  \`\`\`
  Scenario: Verify pom.xml has Java 17
    Tool: Bash
    Preconditions: pom.xml exists
    Steps:
      1. grep -n "java.version" pom.xml
    Expected Result: Line shows <java.version>17</java.version>
    Failure Indicators: Shows 21 instead of 17
    Evidence: grep output showing correct version
  \`\`\`

  **Commit**: YES
  - Message: `fix: downgrade Java version to 17 for system compatibility`
  - Files: `pom.xml`

---

- [ ] 2. Create Flyway migration directory and initial schema

  **What to do**:
  - Create directory: `src/main/resources/db/migration/`
  - Create file: `V1__initial_schema.sql` with basic schema structure
  - Include: users table, subscriptions table (based on package structure)
  - Add indexes for performance

  **Must NOT do**:
  - Don't add real business logic - just placeholder structure
  - Don't run Flyway - just create the file

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Creating a single SQL file with basic DDL
  - **Skills**: None required

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 3, 4)
  - **Blocks**: Task 5 (compile verification)
  - **Blocked By**: None

  **References**:
  - `application.yml:28` - `locations: classpath:db/migration` - where Flyway looks

  **Acceptance Criteria**:
  - [ ] Directory `src/main/resources/db/migration/` exists
  - [ ] File `V1__initial_schema.sql` exists with valid SQL
  - [ ] File contains at least one CREATE TABLE statement
  - [ ] Flyway comment header present (version info)

  **QA Scenarios**:

  \`\`\`
  Scenario: Verify Flyway migration file exists
    Tool: Bash
    Preconditions: None
    Steps:
      1. ls -la src/main/resources/db/migration/
      2. head -5 src/main/resources/db/migration/V1__initial_schema.sql
    Expected Result: Directory exists, V1 file exists, contains "-- Flyway" header
    Failure Indicators: Directory or file missing
    Evidence: ls output and file contents

  Scenario: Verify SQL syntax is valid (basic check)
    Tool: Bash
    Preconditions: V1__initial_schema.sql exists
    Steps:
      1. grep -c "CREATE TABLE" src/main/resources/db/migration/V1__initial_schema.sql
    Expected Result: At least 1 CREATE TABLE found
    Failure Indicators: No CREATE TABLE statements
    Evidence: grep output showing table count >= 1
  \`\`\`

  **Commit**: YES
  - Message: `feat: add initial Flyway migration for database schema`
  - Files: `src/main/resources/db/migration/V1__initial_schema.sql`

---

- [ ] 3. Fix or delete misplaced package-info.java

  **What to do**:
  - Option A (RECOMMENDED): Delete the file at `src/main/java/package-info.java`
  - Option B: Fix the package declaration to `package subscription_manager;`

  **Must NOT do**:
  - Don't leave file with incorrect package declaration
  - Don't add any new content

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Single file deletion or edit
  - **Skills**: None required

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 2, 4)
  - **Blocks**: Task 5 (compile verification)
  - **Blocked By**: None

  **References**:
  - `src/main/java/subscription_manager/package-info.java` - Correct location example
  - `src/main/java/package-info.java` - Current misplaced file

  **Acceptance Criteria**:
  - [ ] Either: File deleted at `src/main/java/package-info.java`
  - [ ] Or: File has correct `package subscription_manager;` declaration
  - [ ] No compile errors related to this file

  **QA Scenarios**:

  \`\`\`
  Scenario: Verify package-info.java is fixed or deleted
    Tool: Bash
    Preconditions: None
    Steps:
      1. cat src/main/java/package-info.java 2>/dev/null || echo "FILE_DELETED"
    Expected Result: Either "FILE_DELETED" or file content with correct package
    Failure Indicators: File exists with wrong package `subscription_manager.src.main.java`
    Evidence: cat output or FILE_DELETED message
  \`\`\`

  **Commit**: YES
  - Message: `fix: remove misplaced package-info.java`
  - Files: `src/main/java/package-info.java` (deleted) or same file (fixed)

---

- [ ] 4. Add Redis configuration to application.yml

  **What to do**:
  - Add Redis connection configuration to `application.yml`
  - Configure localhost:6379 with no password (development default)
  - Add appropriate timeout and pool settings

  **Must NOT do**:
  - Don't change Flyway settings
  - Don't change database configuration

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Simple YAML property addition
  - **Skills**: None required

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 2, 3)
  - **Blocks**: Task 6 (app startup verification)
  - **Blocked By**: None

  **References**:
  - `application.yml` - Current configuration
  - `pom.xml:67-69` - Redis starter dependency

  **Acceptance Criteria**:
  - [ ] `spring.data.redis` section exists in application.yml
  - [ ] Contains host: localhost
  - [ ] Contains port: 6379

  **QA Scenarios**:

  \`\`\`
  Scenario: Verify Redis config exists
    Tool: Bash
    Preconditions: application.yml exists
    Steps:
      1. grep -A5 "redis:" src/main/resources/application.yml || grep -A5 "redis:" application.yml
    Expected Result: Contains host and port configuration
    Failure Indicators: No redis section found
    Evidence: grep output showing redis config

  Scenario: Verify Redis config is valid YAML
    Tool: Bash
    Preconditions: application.yml exists
    Steps:
      1. python3 -c "import yaml; yaml.safe_load(open('src/main/resources/application.yml' if exists else 'application.yml'))" 2>/dev/null || echo "YAML_INVALID"
    Expected Result: No error output (valid YAML)
    Failure Indicators: Python YAML parse error
    Evidence: Error message if YAML is invalid
  \`\`\`

  **Commit**: YES
  - Message: `config: add Redis configuration for cache support`
  - Files: `application.yml`

---

- [ ] 5. Verify Maven compilation succeeds

  **What to do**:
  - Run `./mvnw compile`
  - Verify BUILD SUCCESS
  - Check for zero compilation errors

  **Must NOT do**:
  - Don't fix any additional issues - just verify
  - If errors found, report them for separate fix

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Simple verification command
  - **Skills**: None required

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 2 (sequential after Wave 1)
  - **Blocks**: Task 6 (app startup)
  - **Blocked By**: Tasks 1, 2, 3, 4

  **References**:
  - Maven wrapper: `./mvnw`
  - pom.xml - Build configuration

  **Acceptance Criteria**:
  - [ ] `mvn compile` exits with code 0
  - [ ] Output contains "BUILD SUCCESS"
  - [ ] No "COMPILATION ERROR" in output

  **QA Scenarios**:

  \`\`\`
  Scenario: Verify compilation succeeds
    Tool: Bash
    Preconditions: All Wave 1 tasks completed
    Steps:
      1. export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
      2. cd /mnt/e/subscription-manager/subscription-manager && ./mvnw compile
    Expected Result: "BUILD SUCCESS" in output, exit code 0
    Failure Indicators: "BUILD FAILURE", "COMPILATION ERROR", exit code non-zero
    Evidence: Full mvn output showing success or failure
  \`\`\`

  **Commit**: NO (verification task)

---

- [ ] 6. Verify application startup

  **What to do**:
  - Run `./mvnw spring-boot:run`
  - Wait for startup (up to 60 seconds)
  - Verify no Flyway errors about missing migrations
  - Verify database connection attempt (even if fails due to no DB)

  **Must NOT do**:
  - Don't leave app running - use timeout and stop
  - Don't make changes - just verify

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Verification with timeout
  - **Skills**: None required

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 2 (sequential after Task 5)
  - **Blocks**: None
  - **Blocked By**: Task 5

  **References**:
  - `application.yml` - Spring configuration
  - `pom.xml` - Spring Boot starter configuration

  **Acceptance Criteria**:
  - [ ] No "No migrations found" Flyway error
  - [ ] Application context starts loading
  - [ ] Can be interrupted after 30 seconds with expected errors (Redis, DB)

  **QA Scenarios**:

  \`\`\`
  Scenario: Verify application starts without Flyway errors
    Tool: Bash
    Preconditions: Task 5 passed (compilation)
    Steps:
      1. export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
      2. cd /mnt/e/subscription-manager/subscription-manager && timeout 45 ./mvnw spring-boot:run 2>&1 | head -100
    Expected Result: No "No migrations found" error, application context starts
    Failure Indicators: "No migrations found", Flyway-related errors
    Evidence: mvn output showing startup progress or expected connection errors (not Flyway errors)

  Scenario: Verify application is loadable
    Tool: Bash
    Preconditions: Compilation passed
    Steps:
      1. export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
      2. cd /mnt/e/subscription-manager/subscription-manager && timeout 45 ./mvnw spring-boot:run 2>&1
    Expected Result: Application attempts to start, may fail on DB/Redis but Flyway finds migrations
    Failure Indicators: Flyway error about missing migrations
    Evidence: Startup logs showing Flyway passed
  \`\`\`

  **Commit**: NO (verification task)

---

## Final Verification Wave

- [ ] F1. **All Critical Errors Resolved** — Quick verification
  Verify all 3 critical issues from the original analysis are fixed:
  - Java version = 17 ✓
  - Flyway migration exists ✓
  - No package-info.java errors ✓
  
  Output: `CRITICAL ERRORS [3/3] RESOLVED`

---

## Commit Strategy

- **1**: `fix: downgrade Java version to 17 for system compatibility` — pom.xml
- **2**: `feat: add initial Flyway migration for database schema` — V1__initial_schema.sql
- **3**: `fix: remove misplaced package-info.java` — deleted file
- **4**: `config: add Redis configuration for cache support` — application.yml

---

## Success Criteria

### Verification Commands
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
cd /mnt/e/subscription-manager/subscription-manager
./mvnw compile  # Expected: BUILD SUCCESS
```

### Final Checklist
- [ ] All 3 critical blockers resolved
- [ ] Compilation succeeds without errors
- [ ] Flyway finds migrations
- [ ] No package declaration errors
