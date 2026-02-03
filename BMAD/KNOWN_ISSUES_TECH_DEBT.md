# Known Issues and Technical Debt

## Executive Summary

This document provides an honest assessment of technical debt, risks, and areas requiring improvement in the Directus
codebase. This analysis is based on code inspection, architecture review, and identification of patterns that may impact
maintainability, scalability, or reliability.

**Critical Areas**: Configuration complexity, tight coupling in services layer, missing `.env.example`, test coverage
gaps, permission system complexity.

---

## 1. Configuration and Environment Risks

### 1.1 Missing Root `.env.example` File

**Risk**: HIGH  
**Description**: There is no `.env.example` file at the repository root to guide new developers or deployments.

**Why it matters**:

- New developers must hunt through documentation or source code to discover required environment variables
- Deployment teams lack a template for production configuration
- Increases likelihood of misconfiguration in production
- 199 environment variables defined in `packages/env/src/constants/defaults.ts` with no consolidated reference

**Suggested improvement**:

- Create a comprehensive `.env.example` at repository root
- Include all environment variables with descriptions and example values
- Separate into sections: Required, Optional, Development, Production
- Add inline comments explaining critical variables

### 1.2 Environment Variable Complexity

**Risk**: MEDIUM  
**Description**: 199+ environment variables with complex interdependencies and no validation until runtime.

**Why it matters**:

- Configuration errors only surface when the specific feature is used
- No startup validation for optional features (e.g., `STORAGE_S3_*` variables only validated when S3 is accessed)
- Different database vendors require different sets of variables
- Silent failures possible if optional features are misconfigured

**Suggested improvement**:

- Implement startup validation for all configured features
- Provide clear error messages for missing required variables
- Create configuration presets for common scenarios
- Add environment variable schema validation using Joi/Zod

### 1.3 Database-Specific Configuration Gaps

**Risk**: MEDIUM  
**Description**: Database configuration requirements vary significantly by vendor, but documentation is scattered.

**Why it matters**:

- PostgreSQL requires different variables than MySQL
- Oracle and MSSQL have unique connection requirements
- Connection pool settings have different defaults per database
- No clear guidance on production-ready settings per database

**Suggested improvement**:

- Create database-specific configuration guides
- Provide recommended production settings for each database
- Add validation that checks for vendor-specific required variables
- Document connection pool tuning for each database type

---

## 2. Tight Coupling and Architecture Concerns

### 2.1 ItemsService God Object

**Risk**: HIGH  
**Description**: `api/src/services/items.ts` is 39KB (1240 lines) and handles all CRUD operations for all collections.

**Why it matters**:

- Single point of failure for all item operations
- Difficult to test in isolation
- Hard to extend or modify without affecting all collections
- Performance optimizations require changes to this central class
- Handles permissions, relations, hooks, validation, and database operations in one class

**Suggested improvement**:

- Extract relational logic into separate service
- Separate permission validation into dedicated module
- Consider collection-specific service extensions
- Break down into smaller, focused classes (Single Responsibility Principle)

### 2.2 Permission System Complexity

**Risk**: HIGH  
**Description**: Permission validation is deeply integrated throughout the codebase with complex filter injection and
case-based SQL generation.

**Why it matters**:

- Permission logic in `api/src/permissions/` is spread across 92 files
- Dynamic filter injection makes queries hard to debug
- Performance impact of permission checks is not well-documented
- Difficult to reason about permission behavior in edge cases
- 30+ TODO comments in permission-related code indicate incomplete work

**Example TODOs found**:

- `permissions/modules/fetch-accountability-collection-access`: "Should fields be null, undefined or empty array if no
  access?"
- `permissions/modules/process-ast`: "This can be optimized if there is only one rule"

**Suggested improvement**:

- Document permission evaluation algorithm clearly
- Add performance benchmarks for permission checks
- Simplify permission filter injection
- Create comprehensive test suite for permission edge cases
- Resolve outstanding TODOs

### 2.3 Schema Introspection Dependency

**Risk**: MEDIUM  
**Description**: System relies on runtime database schema introspection, creating tight coupling between database state
and application state.

**Why it matters**:

- Schema changes require application restart or cache invalidation
- No compile-time safety for schema changes
- Database migrations can break running instances
- Schema cache invalidation is complex and error-prone
- Difficult to version control schema changes

**Suggested improvement**:

- Implement schema versioning
- Add schema change detection and validation
- Provide migration rollback capabilities
- Document schema cache invalidation strategy
- Consider schema snapshot exports for version control

---

## 3. Test Coverage Gaps

### 3.1 Missing Integration Tests

**Risk**: HIGH  
**Description**: While unit tests exist (100+ test files found), integration test coverage appears limited.

**Why it matters**:

- Unit tests don't catch integration issues between services
- Database-specific behavior may not be tested across all vendors
- Permission system integration with actual queries is hard to test in isolation
- Real-world workflows (e.g., creating items with relations) may have untested edge cases

**Suggested improvement**:

- Add end-to-end integration tests for critical workflows
- Test against all supported database vendors in CI
- Add performance regression tests
- Test permission system with real database queries

### 3.2 Unimplemented Tests

**Risk**: MEDIUM  
**Description**: Multiple `test.todo()` markers found in codebase indicating planned but unimplemented tests.

**Examples found**:

- `permissions/modules/process-ast/utils/flatten-filter.test.ts`: "Flattens single level and handles underscore in field
  names"
- `utils/validate-user-count-integrity.test.ts`: "unimplemented test"

**Why it matters**:

- Indicates known testing gaps
- Features may lack test coverage
- Regression risk for untested code paths

**Suggested improvement**:

- Implement all `test.todo()` markers
- Add test coverage requirements to CI
- Track test coverage metrics over time

### 3.3 Logger Mocking Issues

**Risk**: LOW  
**Description**: Multiple test files have comment: "reduce to just mock the file when logger is also using useLogger
everywhere @TODO"

**Why it matters**:

- Inconsistent logger usage makes testing harder
- Test setup is more complex than necessary
- Indicates technical debt in logging infrastructure

**Suggested improvement**:

- Standardize logger usage across codebase
- Simplify logger mocking in tests
- Use dependency injection for logger

---

## 4. Scalability Concerns

### 4.1 In-Memory Caching Default

**Risk**: MEDIUM  
**Description**: Default cache store is in-memory (`RATE_LIMITER_STORE: 'memory'`), which doesn't work in multi-instance
deployments.

**Why it matters**:

- Horizontal scaling requires Redis or external cache
- Rate limiting won't work correctly across multiple instances without Redis
- Schema cache won't be shared between instances
- Session storage won't work in load-balanced setups

**Suggested improvement**:

- Document Redis requirement for production deployments
- Add startup warning if in-memory cache is used with multiple instances
- Provide clear migration path from memory to Redis
- Consider making Redis required for production

### 4.2 Unbounded Batch Operations

**Risk**: HIGH  
**Description**: `MAX_BATCH_MUTATION: Infinity` allows unlimited batch operations by default.

**Why it matters**:

- Single request can create/update/delete unlimited items
- No protection against accidental or malicious bulk operations
- Can cause database connection exhaustion
- Memory issues with large payloads
- No rate limiting on batch operations

**Suggested improvement**:

- Set reasonable default limit (e.g., 100 or 1000)
- Document batch operation limits
- Add separate rate limiting for batch operations
- Implement streaming for large batch operations

### 4.3 Query Complexity Limits

**Risk**: MEDIUM  
**Description**: `MAX_RELATIONAL_DEPTH: 10` allows deeply nested queries that can cause performance issues.

**Why it matters**:

- Deep relational queries generate complex SQL joins
- Can cause database performance degradation
- No query cost estimation or timeout
- Potential for denial-of-service via complex queries

**Suggested improvement**:

- Add query complexity scoring
- Implement query timeouts
- Monitor slow queries
- Consider reducing default depth limit
- Add query plan analysis for optimization

---

## 5. Security and Authentication Risks

### 5.1 Default Security Settings

**Risk**: MEDIUM  
**Description**: Several security features are disabled by default:

- `RATE_LIMITER_ENABLED: false`
- `RATE_LIMITER_GLOBAL_ENABLED: false`
- `RATE_LIMITER_EMAIL_ENABLED: false`

**Why it matters**:

- New installations are vulnerable to brute force attacks
- No protection against email flooding
- Requires manual configuration to secure
- Users may not know to enable these features

**Suggested improvement**:

- Enable rate limiting by default
- Provide secure-by-default configuration
- Add security checklist to documentation
- Warn on startup if security features are disabled

### 5.2 Session Token Storage

**Risk**: MEDIUM  
**Description**: Refresh tokens stored in database without clear expiration strategy.

**Why it matters**:

- Refresh tokens can accumulate over time
- No automatic cleanup of expired tokens
- Database bloat from old sessions
- Potential security risk from long-lived tokens

**Suggested improvement**:

- Implement automatic token cleanup
- Add token rotation policy
- Document token lifecycle
- Provide admin tools for token management

---

## 6. Code Quality and Maintainability

### 6.1 Outstanding TODOs

**Risk**: MEDIUM  
**Description**: 30+ TODO comments found in codebase indicating deferred work.

**Critical TODOs**:

- `services/relations.ts`: "@TODO This is a bit of a hack, and might be better of abstracted elsewhere"
- `controllers/permissions.ts`: "TODO fix this at the service level"
- `services/users.ts`: "TODO: translate after there's support for internationalized emails"
- `utils/validate-query.ts`: "@ts-ignore TODO Check which case this is supposed to cover"

**Why it matters**:

- Indicates incomplete features or known issues
- Technical debt accumulation
- Potential bugs or edge cases
- Code that authors knew needed improvement

**Suggested improvement**:

- Create issues for all TODOs
- Prioritize and schedule TODO resolution
- Add context to TODOs (why deferred, impact, etc.)
- Prevent new TODOs without associated issues

### 6.2 TypeScript `@ts-ignore` Usage

**Risk**: LOW  
**Description**: Multiple `@ts-ignore` comments found, bypassing type safety.

**Examples**:

- `websocket/controllers/base.ts`: "TODO Remove once @types/ws has been updated"
- `utils/validate-query.ts`: "TODO Check which case this is supposed to cover"

**Why it matters**:

- Bypasses TypeScript's safety guarantees
- May hide actual type errors
- Technical debt that should be resolved

**Suggested improvement**:

- Replace `@ts-ignore` with `@ts-expect-error` (fails if error is fixed)
- Create issues for each `@ts-ignore`
- Update type definitions where possible
- Document why type ignore is necessary

### 6.3 Unclear Intent in Core Logic

**Risk**: MEDIUM  
**Description**: Comments like "TODO when would this happen?" and "TODO explain this odd case" in critical code paths.

**Examples**:

- `services/items.ts`: "TODO when would this happen?"
- `database/run-ast/utils/merge-with-parent-items.ts`: "TODO explain this odd case"

**Why it matters**:

- Indicates code that even the authors don't fully understand
- Maintenance risk when original authors leave
- Potential bugs in edge cases
- Difficult to debug issues

**Suggested improvement**:

- Document all unclear code paths
- Add comprehensive comments explaining logic
- Create test cases for edge cases
- Refactor unclear code for clarity

---

## 7. Database and Migration Risks

### 7.1 Migration Rollback Support

**Risk**: HIGH  
**Description**: No clear rollback mechanism for database migrations.

**Why it matters**:

- Failed migrations can leave database in inconsistent state
- No easy way to revert schema changes
- Production deployments are risky
- Downtime required for migration failures

**Suggested improvement**:

- Implement migration rollback functionality
- Add migration testing in CI
- Document migration failure recovery procedures
- Provide migration dry-run capability

### 7.2 Database Vendor Differences

**Risk**: MEDIUM  
**Description**: Different SQL dialects handled with vendor-specific code, but testing across all vendors is
challenging.

**Why it matters**:

- Features may work on PostgreSQL but fail on MySQL
- Oracle and MSSQL have unique constraints
- SQLite limitations not well-documented
- Difficult to guarantee consistent behavior

**Suggested improvement**:

- Test all features against all supported databases
- Document vendor-specific limitations
- Consider reducing supported database count
- Add database compatibility matrix to documentation

---

## 8. Extension System Risks

### 8.1 Extension Security

**Risk**: HIGH  
**Description**: Extensions run with full system access and can execute arbitrary code.

**Why it matters**:

- Malicious extensions can compromise entire system
- No sandboxing or permission model for extensions
- Extensions can access all data and APIs
- No extension verification or signing

**Suggested improvement**:

- Implement extension permission model
- Add extension sandboxing
- Provide extension verification/signing
- Document extension security best practices
- Add extension audit logging

### 8.2 Extension Version Compatibility

**Risk**: MEDIUM  
**Description**: No clear extension API versioning or compatibility guarantees.

**Why it matters**:

- Directus updates may break extensions
- No way to declare extension compatibility
- Users may install incompatible extensions
- Extension developers lack stability guarantees

**Suggested improvement**:

- Implement extension API versioning
- Add compatibility checking on installation
- Document breaking changes in API
- Provide extension migration guides

---

## 9. Monitoring and Observability Gaps

### 9.1 Limited Built-in Monitoring

**Risk**: MEDIUM  
**Description**: Metrics endpoint requires explicit enablement (`METRICS_ENABLED=true`), and monitoring is opt-in.

**Why it matters**:

- Production issues hard to diagnose without metrics
- No built-in health checks beyond basic ping
- Performance degradation may go unnoticed
- No alerting capabilities

**Suggested improvement**:

- Enable basic metrics by default
- Add comprehensive health checks
- Provide performance monitoring dashboard
- Document observability best practices
- Add request tracing capabilities

### 9.2 Error Tracking

**Risk**: MEDIUM  
**Description**: Error tracking (Sentry) is optional and requires configuration.

**Why it matters**:

- Production errors may go unnoticed
- No centralized error aggregation
- Difficult to track error trends
- User-reported issues hard to correlate with logs

**Suggested improvement**:

- Provide built-in error aggregation
- Add error rate monitoring
- Implement error alerting
- Document error tracking setup

---

## 10. Documentation and Knowledge Gaps

### 10.1 Missing Architecture Documentation

**Risk**: MEDIUM  
**Description**: While code is well-structured, high-level architecture documentation is limited.

**Why it matters**:

- New developers struggle to understand system design
- Design decisions not documented
- Difficult to onboard new team members
- Risk of architectural drift

**Suggested improvement**:

- Document architectural decisions (ADRs)
- Create system architecture diagrams
- Explain design patterns used
- Document performance characteristics

### 10.2 Production Deployment Guide Gaps

**Risk**: MEDIUM  
**Description**: Production deployment best practices not comprehensively documented.

**Why it matters**:

- Users may deploy with suboptimal configuration
- Security hardening steps not clear
- Scaling strategies not documented
- Backup and recovery procedures unclear

**Suggested improvement**:

- Create comprehensive production deployment guide
- Document scaling strategies
- Provide security hardening checklist
- Add disaster recovery procedures
- Include performance tuning guide

---

## Priority Matrix

| Issue                        | Risk Level | Impact | Effort | Priority     |
| ---------------------------- | ---------- | ------ | ------ | ------------ |
| Missing `.env.example`       | HIGH       | HIGH   | LOW    | **CRITICAL** |
| ItemsService God Object      | HIGH       | HIGH   | HIGH   | **HIGH**     |
| Unbounded Batch Operations   | HIGH       | HIGH   | LOW    | **CRITICAL** |
| Extension Security           | HIGH       | HIGH   | HIGH   | **HIGH**     |
| Permission System Complexity | HIGH       | MEDIUM | HIGH   | **MEDIUM**   |
| Migration Rollback           | HIGH       | HIGH   | MEDIUM | **HIGH**     |
| In-Memory Cache Default      | MEDIUM     | HIGH   | LOW    | **HIGH**     |
| Missing Integration Tests    | HIGH       | MEDIUM | HIGH   | **MEDIUM**   |
| Default Security Settings    | MEDIUM     | HIGH   | LOW    | **HIGH**     |
| Outstanding TODOs            | MEDIUM     | MEDIUM | MEDIUM | **MEDIUM**   |

---

## Recommendations

### Immediate Actions (Next Sprint)

1. Create comprehensive `.env.example` file
2. Set reasonable default for `MAX_BATCH_MUTATION`
3. Enable rate limiting by default
4. Document Redis requirement for production

### Short-term (Next Quarter)

1. Resolve critical TODOs in permission system
2. Add integration test suite
3. Implement migration rollback
4. Create production deployment guide
5. Add extension permission model

### Long-term (Next Year)

1. Refactor ItemsService into smaller components
2. Simplify permission system architecture
3. Implement comprehensive monitoring
4. Add query complexity limits
5. Improve extension security model

---

## Conclusion

Directus is a well-architected system with a solid foundation. The identified issues are typical of a mature,
feature-rich platform. Most risks can be mitigated through configuration, documentation, and incremental refactoring.

**Key Strengths**:

- Modular monorepo structure
- Comprehensive test coverage in many areas
- Flexible extension system
- Multi-database support

**Key Weaknesses**:

- Configuration complexity
- Tight coupling in core services
- Security features disabled by default
- Limited production deployment guidance

Addressing the critical and high-priority items will significantly improve system reliability, security, and
maintainability.
