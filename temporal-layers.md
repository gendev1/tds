# The Three Functional Layers

TDS organizes knowledge across three functional layers, each serving different purposes and audiences. Understanding these layers is crucial for effective TDS implementation.

## Layer 1: Guides Layer (Comments in Code)

### Purpose

Capture immediate context, discoveries, and tactical decisions as they happen.

### Characteristics

-   **Highly contextual**: Tied to specific code location
-   **Immediate**: Written during or right after discovery
-   **Transient**: Expected to change or be promoted
-   **Personal**: Written by/for the developer currently working
-   **Tactical**: Focused on immediate problem-solving

### Comment Types

#### TODO Comments

```go
// TODO: API returning 503s intermittently today
// TODO: Investigate why batch processing times out at 10k records
// TODO: Add retry logic when vendor API rate limits (happens at 3PM daily)
```

**When to use**: Known issues that need attention
**Include**: Specific context, urgency, discovered patterns

#### HACK Comments

```go
// HACK: Working around pagination bug in account service
// HACK: Hardcoding timeout to 45s because vendor API is slow
// HACK: Using deprecated auth method until v2 is stable
```

**When to use**: Temporary workarounds
**Include**: Why the hack exists, what it's working around, when to remove

#### NOTE Comments

```go
// NOTE: This timeout might need adjustment during high load
// NOTE: Deployment requires manual DB migration first
// NOTE: Remember to update this when API changes to v2
```

**When to use**: Important context for future changes
**Include**: What to remember, when it matters, potential impacts

#### LEARNING Comments

```go
// LEARNING: External API expects camelCase, not snake_case
// LEARNING: API rate limits at 100 req/min, not 1000 as documented
// LEARNING: Bulk operations must be < 50k records or silent failure
```

**When to use**: Fresh discoveries that surprised you
**Include**: What was learned, cost of not knowing, context

#### FIXME Comments

```go
// FIXME: This breaks with empty responses from user service
// FIXME: Race condition when multiple requests hit this endpoint
// FIXME: Memory leak in batch processing loop
```

**When to use**: Known bugs that need immediate attention
**Include**: What breaks, under what conditions, impact

### Best Practices

#### Be Specific

```go
// BAD: Generic and unhelpful
// TODO: Fix this bug

// GOOD: Specific and actionable
// TODO: Handle null user_id from auth service (causes 500 on profile page)
```

#### Include Context

```go
// BAD: Missing context
// HACK: Using 30s timeout

// GOOD: Rich context
// HACK: 30s timeout because vendor API is slow (avg 12s response time)
// Remove when they fix infrastructure (promised by Q1 2025)
```

#### Date Time-Sensitive Items

```go
// NOTE: Temporary fix for Q4 rush (Dec 2024) - revert after holidays
// TODO: Remove this workaround after vendor API v2 release (Jan 2025)
```

#### Capture Discoveries

```go
// LEARNING: Payment API requires EST timezone (found during Dec 2024 bug)
// Cost us 3 hours + failed transactions when we used UTC
```

### Promotion Criteria

**Promote to Anchors Layer when:**

-   Learning cost >30 minutes of debugging
-   Insight would help future team members
-   External service quirks discovered
-   Pattern emerges from multiple similar issues

**Keep in Guides Layer when:**

-   Temporary/tactical issues
-   Personal reminders
-   Work-in-progress notes
-   Immediate context only

## Layer 2: Anchors Layer (Anchor Documents)

### Purpose

Preserve institutional knowledge and lessons learned for future reference.

### Characteristics

-   **Structured**: Organized markdown documents
-   **Searchable**: Categorized and findable
-   **Contextual**: Includes discovery date and circumstances
-   **Permanent**: Intended to last and be referenced
-   **Shared**: Written for team consumption

### Organization Structure

```
anchors/
├── domain/              # Business domain knowledge
│   ├── user-permissions.md
│   ├── payment-flows.md
│   └── subscription-logic.md
├── integrations/        # External service learnings
│   ├── payment-api.md
│   ├── email-service.md
│   └── analytics-tracking.md
├── lessons-learned/     # General technical discoveries
│   ├── database-performance.md
│   ├── deployment-gotchas.md
│   └── testing-insights.md
└── README.md           # Navigation and search guide
```

### Anchor Document Template

````markdown
# [Topic] - [Brief Description]

**Discovered**: [Date]
**Context**: [Situation/circumstances]
**Cost**: [Time/impact if not known]
**Last Updated**: [Date]

## The Issue

[Detailed description of the problem/discovery]

## The Solution

[What we learned/how we solved it]

## Code Example

```language
[Relevant code snippet]
```
````

## Impact

-   [Specific consequences of not knowing this]
-   [Benefits of following this approach]
-   [Related issues this prevents]

## Related

-   [Links to related anchors]
-   [Patterns that emerged from this]
-   [Relevant external resources]

````

### Real Examples

#### Integration Anchor
```markdown
# Payment API Timezone Requirements

**Discovered**: 2024-11-20
**Context**: Integration with vendor payment system
**Cost**: 3 hours debugging + failed transactions
**Last Updated**: 2024-11-20

## The Issue
Payment API requires all timestamps in EST timezone.
UTC timestamps are silently rejected with 400 error.
No timezone specified = assumes local server time (dangerous).

## The Solution
Always convert timestamps to EST before sending to payment API.
Include timezone information explicitly in all requests.

## Code Example
```go
func ToPaymentAPITimestamp(t time.Time) string {
    est, _ := time.LoadLocation("America/New_York")
    return t.In(est).Format(time.RFC3339)
}
````

## Impact

-   Failed transactions during timezone confusion
-   Support tickets from confused users
-   3 hours of debugging time
-   Production downtime during busy period

## Related

-   patterns/integrations/timezone-safety.md
-   anchors/integrations/payment-api-general.md

````

#### Domain Knowledge Anchor
```markdown
# User Permission Inheritance

**Discovered**: 2024-10-15
**Context**: Adding role-based access control
**Cost**: 2 days of refactoring + security review
**Last Updated**: 2024-10-15

## The Issue
User permissions are inherited from multiple sources:
1. Direct user permissions
2. Role permissions
3. Group permissions
4. Organization permissions

The inheritance order matters and affects security model.

## The Solution
Permission resolution order (highest to lowest priority):
1. Explicit DENY on user
2. Explicit ALLOW on user
3. Role-based permissions
4. Group-based permissions
5. Organization defaults

## Code Example
```go
func ResolveUserPermissions(userID string) PermissionSet {
    // Implementation following inheritance order
}
````

## Impact

-   Security vulnerabilities if inheritance is wrong
-   Confusing user experience with inconsistent access
-   Performance issues from N+1 permission queries
-   Difficulty debugging access issues

## Related

-   patterns/security/permission-checking.md
-   anchors/domain/role-management.md

```

### Anchor Maintenance

#### When to Update
- Bug fixes related to the anchor
- New discoveries that expand understanding
- Changes in external systems referenced
- Team feedback on clarity/completeness

#### Review Schedule
- **Monthly**: Check for stale information
- **Quarterly**: Validate external references
- **Annually**: Consider promotion to patterns

## Layer 3: Patterns Layer (Patterns)

### Purpose
Codify proven solutions and safeguards to prevent known failure modes.

### Characteristics
- **Reusable**: Can be applied to multiple situations
- **Proven**: Based on multiple real-world uses
- **Protective**: Prevents known failure modes
- **Actionable**: Includes concrete implementation guidance

### Organization Structure

```

patterns/
├── error-handling/
│ ├── retry-logic.md
│ ├── timeout-handling.md
│ └── circuit-breaker.md
├── integrations/
│ ├── api-client-safety.md
│ ├── webhook-verification.md
│ └── timezone-handling.md
├── testing/
│ ├── integration-test-setup.md
│ ├── mock-strategies.md
│ └── test-data-management.md
└── README.md

````

### Pattern Document Template

```markdown
# [Pattern Name]

## Problem
[What problem this pattern solves]

## Context
[When/where this pattern applies]

## Solution
[The pattern implementation]

## Implementation
```language
[Complete code example]
````

## Usage

[How to use the pattern]

## Prevents

-   [Specific failure modes prevented]
-   [Common mistakes avoided]
-   [Performance issues addressed]

## Variations

[Alternative implementations]

## Related Patterns

[Links to related patterns]

## Derived From

[Anchors that led to this pattern]

````

### Real Examples

#### Integration Pattern
```markdown
# API Client Safety Pattern

## Problem
External API calls can fail in many ways: timeouts, rate limits, network issues, server errors. Without proper safeguards, these failures cascade through your system.

## Context
Any integration with external services, especially in production systems where reliability matters.

## Solution
Implement defensive API client with timeouts, retries, circuit breaker, and graceful degradation.

## Implementation
```go
type SafeAPIClient struct {
    baseURL    string
    timeout    time.Duration
    retryCount int
    circuitBreaker *CircuitBreaker
}

func (c *SafeAPIClient) Call(ctx context.Context, req APIRequest) (*APIResponse, error) {
    // Timeout protection
    ctx, cancel := context.WithTimeout(ctx, c.timeout)
    defer cancel()

    // Circuit breaker protection
    if c.circuitBreaker.IsOpen() {
        return nil, ErrCircuitBreakerOpen
    }

    // Retry logic
    for attempt := 0; attempt <= c.retryCount; attempt++ {
        resp, err := c.makeRequest(ctx, req)
        if err == nil {
            c.circuitBreaker.RecordSuccess()
            return resp, nil
        }

        if !c.isRetryableError(err) {
            break
        }

        time.Sleep(c.backoffDuration(attempt))
    }

    c.circuitBreaker.RecordFailure()
    return nil, err
}
````

## Usage

```go
client := &SafeAPIClient{
    baseURL:    "https://api.vendor.com",
    timeout:    30 * time.Second,
    retryCount: 3,
    circuitBreaker: NewCircuitBreaker(5, time.Minute),
}

resp, err := client.Call(ctx, request)
if err != nil {
    // Handle failure gracefully
    return fallbackResponse()
}
```

## Prevents

-   Cascading failures from external service outages
-   Hanging requests from network timeouts
-   Rate limit exhaustion from aggressive retries
-   System instability from external dependencies

## Variations

-   Different retry strategies (exponential backoff, jitter)
-   Different circuit breaker configurations
-   Async/queue-based fallbacks
-   Caching for read operations

## Related Patterns

-   patterns/error-handling/timeout-handling.md
-   patterns/error-handling/circuit-breaker.md

## Derived From

-   anchors/integrations/payment-api-failures.md
-   anchors/lessons-learned/external-service-outages.md

```

### Pattern Evolution

#### From Anchor to Pattern
1. **Multiple similar anchors**: Same type of solution appears repeatedly
2. **Proven effectiveness**: Solution has worked in multiple contexts
3. **Generalizable**: Can be abstracted beyond specific use case
4. **Preventive value**: Actively prevents known failure modes

#### Pattern Maturity Levels
- **Emerging**: First extraction from anchors
- **Established**: Used successfully in multiple contexts
- **Mature**: Widely adopted with known variations
- **Standard**: Default approach for common scenarios

## Layer Interactions

### Promotion Flow
```

PRESENT (Comments)
↓ [Weekly Review]
PAST (Anchors)
↓ [Monthly/Quarterly Review]
PROTECTION (Patterns)
↓ [Ongoing]
CODE (Implementation)

```

### Feedback Loop
```

PROTECTION → PRESENT: Patterns guide current implementation
PAST → PRESENT: Anchors provide context for current work
PRESENT → PAST: New discoveries update institutional knowledge

````

### Cross-Layer References

#### Comments Reference Anchors
```go
// See anchors/integrations/payment-api.md for timezone requirements
timestamp := ToPaymentAPITimestamp(time.Now())
````

#### Anchors Reference Patterns

```markdown
## Related Patterns

-   patterns/integrations/api-client-safety.md
```

#### Patterns Reference Anchors

```markdown
## Derived From

-   anchors/integrations/payment-api-failures.md
-   anchors/lessons-learned/timeout-debugging.md
```

## Layer Management

### Cleanup Strategies

#### Guides Layer Cleanup

-   Remove comments promoted to anchors
-   Update stale TODOs
-   Archive completed HACKs
-   Promote valuable LEARNINGs

#### Anchors Layer Maintenance

-   Update anchors with new discoveries
-   Merge similar anchors
-   Archive outdated information
-   Promote patterns to patterns layer

#### Patterns Layer Evolution

-   Refine patterns based on usage
-   Create variations for different contexts
-   Merge similar patterns
-   Deprecate patterns replaced by better solutions

---

**Remember**: The three layers work together as a system. Present captures, Past preserves, Protection prevents. Each layer has its purpose and proper timing.
