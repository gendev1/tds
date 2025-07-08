# Knowledge Flow in TDS

The Temporal Documentation System creates a pipeline where knowledge naturally flows from immediate discovery to long-term institutional memory. Understanding this flow is crucial for effective TDS implementation.

## The Knowledge Flow Pipeline

```
1. DISCOVER (Guides Layer)
   ↓
2. LEARN (Promotion to Anchors Layer)
   ↓
3. PROTECT (Pattern Extraction)
   ↓
4. CLEAN (Comment Cleanup)
```

## Phase 1: Discovery (Guides Layer)

### What Happens

Developers encounter surprises, solve problems, and make discoveries during normal development work.

### Capture Mechanisms

```go
// Immediate capture during development
// LEARNING: API rate limits at 100 req/min, not 1000 as documented
// TODO: Add exponential backoff for retry logic
// HACK: Hardcoding 45s timeout because vendor API is slow
```

### Discovery Triggers

-   **Debugging sessions**: "Why is this failing?"
-   **Integration work**: "How does this external service work?"
-   **Performance issues**: "Why is this slow?"
-   **Error investigation**: "What's causing this exception?"
-   **Code review**: "Why was this implemented this way?"

### Types of Discoveries

1. **Surprise findings**: API behaves differently than expected
2. **Workaround solutions**: Temporary fixes that work
3. **Performance insights**: What's fast/slow and why
4. **Integration quirks**: External service peculiarities
5. **Business logic nuances**: Domain-specific rules

### Discovery Examples

```go
// Surprise finding
// LEARNING: Payment API silently drops records > 50k batch size

// Workaround solution
// HACK: Using deprecated /v1/auth until v2 is stable (broken since Nov 2024)

// Performance insight
// NOTE: This query is slow with >10k users, consider pagination

// Integration quirk
// LEARNING: Webhook signature includes timestamp, expires after 5min

// Business logic nuance
// NOTE: Free tier users get 30-day trial, paid users get 90-day
```

## Phase 2: Learning (Promotion to Anchors Layer)

### When to Promote

**Promote discoveries to anchors when:**

-   Learning cost >30 minutes of debugging time
-   Insight would help future team members
-   External service quirks discovered
-   Pattern emerges from multiple similar issues
-   Business logic is complex or non-obvious

### Promotion Process

#### Weekly Review

```bash
# Find learnings from the week
grep -r "LEARNING:" --include="*.go" . | grep -v "/vendor/"

# Evaluate each learning:
# - Did this cost significant time?
# - Would this help teammates?
# - Is this likely to happen again?
```

#### Create Anchor Document

```markdown
# Payment API Batch Limits

**Discovered**: 2024-11-20
**Context**: Processing large customer imports
**Cost**: 4 hours debugging + data loss

## The Issue

Payment API silently drops records beyond 50k batch size.
No error returned, just processes first 50k records.
```

#### Update Original Comment

```go
// BEFORE: Discovery comment
// LEARNING: Payment API silently drops records > 50k batch size

// AFTER: Reference to anchor
// See anchors/integrations/payment-api-limits.md for batch size limits
const MAX_BATCH_SIZE = 45000 // Safe buffer under 50k limit
```

### Promotion Criteria

#### High-Value Promotions

-   **Time-intensive discoveries**: >30 minutes to figure out
-   **Recurring patterns**: Same issue across multiple areas
-   **Team knowledge gaps**: Information others would need
-   **External service quirks**: Undocumented API behaviors
-   **Business rule complexity**: Non-obvious domain logic

#### Low-Value Promotions (Avoid)

-   **Obvious solutions**: Standard programming patterns
-   **Personal preferences**: Coding style choices
-   **Temporary issues**: One-time problems
-   **Framework basics**: Well-documented standard usage

### Promotion Examples

#### Good Promotion

```go
// Discovery comment
// LEARNING: User permissions are inherited from role -> group -> org
// Order matters: user DENY overrides role ALLOW

// Becomes anchor: anchors/domain/user-permissions.md
```

#### Poor Promotion

```go
// Discovery comment
// LEARNING: Using fmt.Sprintf for string formatting

// Should stay as comment - this is basic Go knowledge
```

## Phase 3: Protection (Pattern Extraction)

### Pattern Identification

Look for repeated solutions across multiple anchors:

```bash
# Find common themes in anchors
grep -r "timeout\|rate limit" anchors/
grep -r "retry\|circuit breaker" anchors/
grep -r "timezone\|timestamp" anchors/
```

### Pattern Extraction Process

#### 1. Identify Repetition

```markdown
# Multiple anchors about API timeouts:

anchors/integrations/payment-api-timeout.md
anchors/integrations/shipping-api-timeout.md
anchors/integrations/email-service-timeout.md
```

#### 2. Abstract Common Solution

```go
// Common pattern across all integrations
type SafeAPIClient struct {
    timeout    time.Duration
    retryCount int
    backoff    BackoffStrategy
}
```

#### 3. Create Pattern Document

```markdown
# API Client Safety Pattern

## Problem

External API calls fail in predictable ways: timeouts, rate limits, network issues.

## Solution

Implement defensive API client with configurable timeouts, retries, and backoff.

## Implementation

[Complete code example]
```

#### 4. Update Anchors

```markdown
# In anchor documents

## Related Patterns

-   patterns/integrations/api-client-safety.md
```

### Pattern Evolution

#### Emerging Patterns

-   **First extraction** from 2-3 similar anchors
-   **Experimental** - not yet proven in multiple contexts
-   **Specific** - addresses narrow use case

#### Mature Patterns

-   **Proven** in 5+ different contexts
-   **General** - applicable across multiple domains
-   **Stable** - implementation settled

#### Standard Patterns

-   **Default choice** for common scenarios
-   **Team consensus** on approach
-   **Documented variations** for edge cases

## Phase 4: Cleanup (Comment Maintenance)

### Cleanup Triggers

-   **Anchor creation**: Remove/update promoted comments
-   **Pattern implementation**: Update comments to reference patterns
-   **Issue resolution**: Clean up TODO/FIXME comments
-   **Stale information**: Remove outdated context

### Cleanup Process

#### After Anchor Creation

```go
// BEFORE: Raw discovery
// LEARNING: Payment API requires EST timezone (costs 3 hours if wrong)

// AFTER: Clean reference
// See anchors/integrations/payment-api-timezone.md
timestamp := ToPaymentAPITimestamp(time.Now())
```

#### After Pattern Implementation

```go
// BEFORE: Specific solution
// HACK: Custom retry logic for flaky vendor API

// AFTER: Pattern reference
// Using API client safety pattern (see patterns/integrations/api-client-safety.md)
client := NewSafeAPIClient(vendorURL, DefaultRetryConfig)
```

#### TODO/FIXME Resolution

```go
// BEFORE: Active TODO
// TODO: Add retry logic for payment API failures

// AFTER: Implementation complete
// Payment API client with retry logic (see patterns/integrations/api-client-safety.md)
```

### Cleanup Strategies

#### Weekly Cleanup

-   Remove comments for issues that were resolved
-   Update promoted LEARNINGs to reference anchors
-   Clean up completed TODOs and HACKs

#### Monthly Cleanup

-   Review anchor references for accuracy
-   Update pattern references in code
-   Archive outdated comments

#### Quarterly Cleanup

-   Major comment audit
-   Pattern consolidation
-   Anchor reorganization

## Flow Optimization

### Accelerating Discovery

-   **Pair programming**: More eyes catch more surprises
-   **Code reviews**: Fresh perspective reveals assumptions
-   **Debugging sessions**: Systematic surprise capture
-   **Integration work**: High-discovery activity

### Improving Promotion

-   **Regular reviews**: Weekly learning evaluation
-   **Team discussions**: Collective wisdom on what to promote
-   **Clear criteria**: >30 minutes = promote
-   **Template usage**: Consistent anchor format

### Enhancing Protection

-   **Pattern discussions**: Team consensus on solutions
-   **Usage tracking**: Monitor pattern adoption
-   **Regular updates**: Keep patterns current
-   **Variation documentation**: Handle edge cases

### Streamlining Cleanup

-   **Automated searches**: Scripts to find promotable content
-   **Review reminders**: Calendar events for cleanup
-   **Link verification**: Ensure references are accurate
-   **Archive strategy**: Remove truly outdated content

## Flow Anti-Patterns

### Discovery Anti-Patterns

-   **Silent suffering**: Not capturing learnings
-   **Vague comments**: "TODO: Fix this" without context
-   **Assumption documentation**: Documenting obvious things

### Promotion Anti-Patterns

-   **Over-promotion**: Anchoring trivial discoveries
-   **Under-promotion**: Leaving valuable insights in comments
-   **Inconsistent promotion**: No clear criteria

### Protection Anti-Patterns

-   **Premature patterns**: Creating patterns before proving value
-   **Pattern paralysis**: Over-engineering solutions
-   **Stale patterns**: Not updating based on usage

### Cleanup Anti-Patterns

-   **Cleanup neglect**: Letting comments accumulate
-   **Over-cleanup**: Removing valuable context
-   **Broken references**: Links to non-existent anchors

## Measuring Flow Health

### Discovery Metrics

-   **LEARNING comments per week**: Insight capture rate
-   **Discovery diversity**: Different types of surprises
-   **Team participation**: Multiple people discovering

### Promotion Metrics

-   **Promotion rate**: LEARNINGs → anchors per week
-   **Promotion quality**: Anchors referenced by team
-   **Promotion consistency**: Clear criteria applied

### Protection Metrics

-   **Pattern creation rate**: Anchors → patterns
-   **Pattern usage**: References in code
-   **Pattern effectiveness**: Issues prevented

### Cleanup Metrics

-   **Comment freshness**: Age of TODO/HACK comments
-   **Reference accuracy**: Working links to anchors
-   **Comment density**: Signal vs noise ratio

## Flow Automation

### Discovery Automation

```bash
# Weekly discovery report
grep -r "LEARNING:" --include="*.go" . | sort | uniq
```

### Promotion Automation

```bash
# Find high-value learnings
grep -r "LEARNING:" --include="*.go" . | grep -E "(hours|cost|debug|failed)"
```

### Protection Automation

```bash
# Find pattern candidates
grep -r "Similar to" anchors/ | head -10
```

### Cleanup Automation

```bash
# Find stale TODOs
grep -r "TODO:" --include="*.go" . | grep -E "(2023|2024)"
```

---

**Remember**: The knowledge flow is a living system. It should feel natural and add value, not create bureaucracy. Start simple and evolve based on what works for your team.
