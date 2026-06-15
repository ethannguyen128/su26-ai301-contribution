# Contribution [#]: [Improvement] Catch-path request dereference can mask original exceptions in REST handlers

**Contribution Number:** 1 
**Student:** Ethan Nguyen 
**Issue:** (https://github.com/apache/gravitino/issues/10172) 
**Status:** Phase 1 Complete

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

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
