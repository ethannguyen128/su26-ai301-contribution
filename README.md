# Contribution [1]: [Improvement] Catch-path request dereference can mask original exceptions in REST handlers

**Contribution Number:** 1 
**Student:** Ethan Nguyen 
**Issue:** (https://github.com/apache/gravitino/issues/10172) 
**Status:** Phase 4 Complete

---

## Why I Chose This Issue

It matches the languages I am comfortable with and hope to work with more in the future.

---

## Understanding the Issue

### Problem Description

Multiple REST handler catch block reference request (e.g. request.getName()) without checking if it is null. When it is null, a primary exception occurs and the catch throws a secondary NullPointerException that hides what the original error was.

### Expected Behavior

The original exception should be preserved and returned through the ExceptionHandlers. There should not be any request.get*() calls in the catch blocks.

### Current Behavior

The actual error is masked by the secondary NullPointerException which makes it difficult to know what the root cause of the error was.

### Affected Components

16 REST handler files under server/src/main/java/org/apache/gravitino/server/web/rest/:
TableOperations.java, FilesetOperations.java, FunctionOperations.java, ModelOperations.java, SchemaOperations.java, MetalakeOperations.java, CatalogOperations.java, UserOperations.java, GroupOperations.java, RoleOperations.java, PolicyOperations.java, TagOperations.java, TopicOperations.java, PermissionOperations.java, StatisticOperations.java, JobOperations.java

---

## Reproduction Process

### Environment Setup

Cloned the fork of apache/gravitino locally using VS Code. No build was required to reproduce this issue as the bug is directly visible in the source code. Attempted to run .\gradlew build -x test but encountered PowerShell execution policy restrictions on Windows

### Steps to Reproduce

1. Open server/src/main/java/org/apache/gravitino/server/web/rest/TableOperations.java
2. Navigate to the createTable method and find the catch (Exception e) block (line 157)
3. Observe that request.getName() is called inside the catch block with no null check on request
4. If request is null when the primary exception occurs, this line throws a secondary NullPointerException that hides the original error
5. The same pattern is repeated in 15 other files listed in the issue scope

### Reproduction Evidence

- **Commit showing reproduction: (https://github.com/ethannguyen128/gravitino/tree/fix-issue-10172)
- **Screenshots/logs:**  bug is visible directly in source code
- **My findings:** The catch blocks in 16 REST handler files dereference request.get*() without null checks. When request is null, a secondary NullPointerException is thrown inside the catch block, masking the original exception and making debugging very difficult.

---

## Solution Approach

### Analysis

The root cause is that catch blocks in 16 REST handler files call request.get*() methods (e.g. request.getName(), request.getRoleNames()) without first checking if request is null. Since request can be null when a REST call fails early, the catch block throws a secondary NullPointerException that replaces the original exception in the error context, making the real failure invisible.

### Proposed Solution

Extract the needed value from request into a local variable before the try block, with a null-safe fallback. The catch block then uses the precomputed variable instead of touching request directly.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Multiple REST handler catch blocks dereference request without null checks. When request is null, a secondary NullPointerException masks the real failure.

**Match:** The fix follows a pattern already used in the codebase, extracting identifiers before the try block so catch blocks never need to access the request object.

**Plan:** 
1. For up to 16 in-scope files, find the catch block that calls request.get*()
2. Extract the value into a local variable before the try block:
String name = request != null ? request.getName() : "unknown";
3. Replace request.getName() in the catch block with the precomputed name variable
4. Add/adjust unit tests to verify the catch path is stable when request is null

**Implement:** (https://github.com/ethannguyen128/gravitino/tree/fix-issue-10172)

**Review:** Verify changes follow CONTRIBUTING.md conventions, no endpoint APIs changed, no ExceptionHandlers behavior altered, and commit message follows project format.

**Evaluate:** I will run .\gradlew test -PskipITs and confirm all tests pass. Add a unit test per affected file that sends a null request body and  asserts the response is not a NullPointerException.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: **14 create/add/register handlers** (Table, Fileset, Function, Model, Schema, Catalog `create` + `testConnection`, User, Group, Role, Policy, Tag, Topic, Job): assert HTTP 500 (INTERNAL_SERVER_ERROR) with type `RuntimeException` — the primary exception is now marshaled cleanly instead of being masked by an NPE.
- [ ] Test case 2: **PermissionOperations** grant path (PUT): same assertion.
- [ ] Test case 3: **StatisticOperations** `dropStatistics`: same assertion.
- [ ] Test case 4: **MetalakeOperations** `create`: asserts HTTP 400 (BAD_REQUEST) / `IllegalArgumentException`. This handler already guards against a null body explicitly, so it returns a clean 400; the test documents that it never exposes an NPE.

### Integration Tests

N/A — the change is exercised through JUnit/Jersey resource tests (`Test*Operations`); no integration test is required.

### Manual Testing
After resolving the PowerShell execution-policy restriction noted in Environment Setup, I ran the build and targeted tests locally:
- `./gradlew :server:spotlessApply` — formatting clean.
- `./gradlew :server:test -PskipITs --tests "org.apache.gravitino.server.web.rest.Test*Operations"`: **172 tests, all passing.**
- The first run surfaced one failure: my MetalakeOperations test incorrectly expected a 500. Investigating showed `createMetalake` already has an explicit null-body guard returning 400; I corrected the test assertion and reverted my unnecessary edit to that handler.


---

## Implementation Notes

### Week [3] Progress
Implemented the fix across all in-scope handlers using the planned pattern, extracted a null-safe identifier into a local variable before the `try`, and used it in the catch path instead of dereferencing `request`:

### Week [4] Progress

Submitted my first PR for the first issue.

### Code Changes

- **Files modified:** - **Source files modified (15):** TableOperations, FilesetOperations, FunctionOperations, ModelOperations, SchemaOperations, CatalogOperations, UserOperations, GroupOperations, RoleOperations, PolicyOperations, TagOperations, TopicOperations, PermissionOperations, StatisticOperations, JobOperations. MetalakeOperations was in scope but already guarded against null — no source change. In StatisticOperations only `dropStatistics` needed the fix; the update / partition-update / partition-drop paths already route through null-safe helpers. Test files modified (16): the matching `Test*Operations` classes.

- **Key commits:** (https://github.com/ethannguyen128/gravitino/commit/d85947d36957d6f61083e93af6dd90d405b528ac)
- **Approach decisions:** I chose to do a single atomic commit containing the fix and its tests, since the project squash-merges PRs and a reviewer expects the fix and its coverage together.


---

## Pull Request

**PR Link:** https://github.com/apache/gravitino/pull/11816

**PR Description:** Fixes a bug in 15 REST handlers where dereferencing request inside catch blocks threw a secondary NullPointerException that masked the original exception when a null request body was sent. The fix precomputes a null-safe identifier before each try block so the real root cause is always preserved and correctly surfaced to the client.

**Maintainer Feedback:**
- 6/28/: No Feedback received yet 

**Status:** Awiting Review

---

## Learnings & Reflections

### Technical Skills Gained

I better learned to reason about the catch block as its own execution path and apply the null-safe ternary guard to hoist identifiers out of request before the try. Also picked up Jersey/JUnit resource testing and that posting a null body and asserting the response is not a NullPointerException.

### Challenges Overcome

PowerShell's execution-policy restriction initially blocked .\gradlew, so I couldn't build right away. My first Metalake test also failed because I wrongly expected a 500.

### What I'd Do Differently Next Time

Next time I would get a green build running first so I can test incrementally instead of finding wrong assumptions late. I'd also read each handler's existing null-handling before writing its test.
---

## Resources Used

- No external resources used.
