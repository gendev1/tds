# TDS Best Practices

This guide captures proven patterns, anti-patterns, and lessons learned from TDS implementations across different teams and projects.

## Implementation Patterns

### The Gradual Adoption Pattern

**Problem**: Teams try to implement full TDS all at once and get overwhelmed.

**Solution**: Phase implementation over 3-6 months.

```
Week 1-2: Just meaningful comments
Week 3-6: Add weekly review cycle
Month 2: Create first anchors
Month 3: Extract first patterns
Month 4+: Refine and optimize
```

**Why it works**: Builds habits gradually, reduces cognitive load, allows natural evolution.

### The Champion Pattern

**Problem**: TDS adoption fails without consistent advocacy.

**Solution**: Designate 1-2 team members as TDS champions.

**Champion responsibilities**:

-   Run weekly TDS reviews
-   Help teammates write good comments
-   Promote valuable learnings to anchors
-   Evangelize TDS benefits

**Why it works**: Ensures consistency, provides expertise, maintains momentum.

### The Context-First Pattern

**Problem**: Teams focus on process over value.

**Solution**: Prioritize capturing context over following rules.

```go
// GOOD: Rich context
// LEARNING: Payment API requires EST timezone (found during Dec 2024 outage)
// Cost us 3 hours + 50 failed transactions when we used UTC
// Vendor confirmed this is undocumented requirement

// BAD: Process-focused
// LEARNING: Timezone issue fixed (promoted to anchor)
```

**Why it works**: Context is the real value, process is just the delivery mechanism.

## Comment Writing Patterns

### The Surprise Pattern

**Problem**: Developers document expected behavior instead of surprises.

**Solution**: Focus on what surprised you, not what you expected.

```go
// GOOD: Captures surprise
// LEARNING: API returns 200 for invalid requests, check response body for errors

// BAD: Documents expected behavior
// LEARNING: API returns JSON responses
```

**Why it works**: Surprises are what cause bugs and wasted time.

### The Cost Pattern

**Problem**: Learnings without cost context don't get prioritized.

**Solution**: Always include the cost of not knowing.

```go
// GOOD: Includes cost
// LEARNING: Bulk operations fail silently over 50k records
// Cost: 4 hours debugging + data loss for 10k customers

// BAD: No cost context
// LEARNING: Bulk operations have limits
```

**Why it works**: Cost justifies the effort to remember and share.

### The Temporal Pattern

**Problem**: Comments become stale without temporal context.

**Solution**: Include when the information was discovered and might change.

```go
// GOOD: Temporal context
// HACK: Using v1 auth until v2 is stable (broken since Nov 2024, fix promised Q1 2025)

// BAD: No temporal context
// HACK: Using old auth method
```

**Why it works**: Helps determine when comments need updating.

## Anchor Creation Patterns

### The Template Pattern

**Problem**: Inconsistent anchor format makes knowledge hard to find.

**Solution**: Use consistent template for all anchors.

```markdown
# [Topic] - [Brief Description]

**Discovered**: [Date]
**Context**: [Situation/circumstances]
**Cost**: [Time/impact if not known]

## The Issue

[What was the problem/discovery]

## The Solution

[What was learned/how it was solved]

## Impact

-   [Consequences of not knowing]
-   [Benefits of following approach]
```

**Why it works**: Consistency improves findability and comprehension.

### The Searchable Pattern

**Problem**: Anchors can't be found when needed.

**Solution**: Use descriptive titles and include search keywords.

```markdown
# Good titles:

-   payment-api-timezone-requirements.md
-   user-permission-inheritance-order.md
-   batch-processing-size-limits.md

# Bad titles:

-   api-issue.md
-   permission-bug.md
-   batch-problem.md
```

**Why it works**: Descriptive names are easier to find and understand.

### The Cross-Reference Pattern

**Problem**: Related knowledge is scattered and hard to connect.

**Solution**: Link related anchors, patterns, and code.

```markdown
## Related

-   anchors/integrations/payment-api-general.md
-   patterns/integrations/timezone-safety.md
-   Code: `utils/payment.go:45`
```

**Why it works**: Creates knowledge network instead of isolated facts.

## Pattern Development Patterns

### The Proven Pattern

**Problem**: Patterns created too early become premature abstractions.

**Solution**: Only create patterns after 3+ successful uses.

```
First use: Specific solution in code
Second use: Similar solution with variations
Third use: Extract common pattern
```

**Why it works**: Ensures patterns are truly reusable and battle-tested.

### The Evolution Pattern

**Problem**: Patterns become stale as systems evolve.

**Solution**: Regularly update patterns based on usage.

```markdown
## Version History

-   v1.0: Initial timeout pattern
-   v1.1: Added exponential backoff
-   v1.2: Added circuit breaker integration
```

**Why it works**: Keeps patterns current and acknowledges evolution.

## Anti-Patterns to Avoid

### The Documentation Theater Anti-Pattern

**Problem**: Creating TDS artifacts for compliance rather than value.

**Symptoms**:

-   Anchors that nobody references
-   Patterns that nobody uses
-   Comments that add no context

**Solution**: Focus on value, not volume. Better to have 5 valuable anchors than 50 useless ones.

### The Perfect Process Anti-Pattern

**Problem**: Trying to create perfect TDS process from day one.

**Symptoms**:

-   Complex promotion rules
-   Mandatory documentation for everything
-   Process paralysis

**Solution**: Start simple, evolve based on what works.

### The Hoarding Anti-Pattern

**Problem**: Keeping all knowledge in comments instead of promoting.

**Symptoms**:

-   LEARNING comments from months ago
-   No anchor creation
-   Knowledge trapped in individual files

**Solution**: Regular promotion cycles and clear promotion criteria.

### The Over-Engineering Anti-Pattern

**Problem**: Building complex tooling before proving TDS value.

**Symptoms**:

-   Complex automation scripts
-   Elaborate metrics dashboards
-   Process overhead exceeding value

**Solution**: Start with simple bash scripts, add complexity as needed.

### The Stale Knowledge Anti-Pattern

**Problem**: Old knowledge pollutes current understanding.

**Symptoms**:

-   Outdated comments in code
-   Stale anchor references
-   Obsolete patterns

**Solution**: Regular cleanup cycles and clear expiration policies.

## Team Patterns

### The Distributed Learning Pattern

**Problem**: Knowledge concentrated in few team members.

**Solution**: Encourage everyone to contribute LEARNING comments.

**Implementation**:

-   Rotate who runs TDS reviews
-   Celebrate learning contributions
-   Make it safe to document mistakes

**Why it works**: Distributes knowledge across team, reduces bus factor.

### The Onboarding Pattern

**Problem**: New team members can't access institutional knowledge.

**Solution**: Use TDS for onboarding documentation.

**Implementation**:

-   Create onboarding anchor with key concepts
-   Reference patterns in code reviews
-   Use TDS search for common questions

**Why it works**: New members get context, not just code.

### The Cross-Team Pattern

**Problem**: Knowledge silos between teams.

**Solution**: Share relevant anchors across team boundaries.

**Implementation**:

-   Common anchor categories
-   Cross-team pattern library
-   Shared TDS practices

**Why it works**: Reduces duplicate learning across organization.

## Quality Patterns

### The Actionable Pattern

**Problem**: Knowledge that can't be acted upon.

**Solution**: Make all TDS artifacts actionable.

```markdown
# GOOD: Actionable

## Solution

Always set timeout to 30s for vendor API calls.
Use exponential backoff starting at 1s.

# BAD: Not actionable

## Solution

The API is unreliable and slow.
```

**Why it works**: Actionable knowledge prevents repeated issues.

### The Specific Pattern

**Problem**: Vague knowledge that doesn't help.

**Solution**: Be specific about contexts, numbers, and conditions.

```go
// GOOD: Specific
// LEARNING: Batch size >10k causes timeout (tested with 15k records)
// API limit appears to be around 12k based on error messages

// BAD: Vague
// LEARNING: Large batches cause problems
```

**Why it works**: Specific knowledge is more valuable and actionable.

### The Validated Pattern

**Problem**: Knowledge that turns out to be wrong.

**Solution**: Include how the knowledge was validated.

```markdown
## Validation

-   Tested with production data on 2024-11-20
-   Confirmed with vendor support ticket #12345
-   Reproduced in staging environment
```

**Why it works**: Builds confidence in the knowledge.

## Maintenance Patterns

### The Lifecycle Pattern

**Problem**: TDS artifacts live forever without review.

**Solution**: Define lifecycle for different types of knowledge.

```
Comments: Review weekly
Anchors: Review monthly
Patterns: Review quarterly
```

**Why it works**: Prevents knowledge decay and maintains relevance.

### The Cleanup Pattern

**Problem**: Accumulating cruft makes TDS harder to use.

**Solution**: Regular cleanup cycles with clear criteria.

```bash
# Monthly cleanup checklist
- [ ] Remove resolved TODOs
- [ ] Update promoted LEARNINGs
- [ ] Archive outdated anchors
- [ ] Fix broken references
```

**Why it works**: Maintains signal-to-noise ratio.

### The Metrics Pattern

**Problem**: No way to measure TDS effectiveness.

**Solution**: Track simple metrics that matter.

```
- LEARNINGs promoted per week
- Anchors referenced in code
- Patterns preventing issues
- Team satisfaction with TDS
```

**Why it works**: Measurement enables improvement.

## Success Patterns

### The Compound Value Pattern

**Problem**: TDS value is hard to see initially.

**Solution**: Expect compound returns over time.

```
Month 1: Individual benefit from better comments
Month 3: Team benefit from shared anchors
Month 6: Organizational benefit from patterns
Year 1: Cultural shift toward learning
```

**Why it works**: Sets proper expectations for value realization.

### The Cultural Pattern

**Problem**: TDS remains a process instead of becoming culture.

**Solution**: Embed TDS into how the team thinks about work.

**Signs of success**:

-   Team naturally asks "Should this be an anchor?"
-   Patterns become default approaches
-   New members adopt TDS without training

**Why it works**: Cultural adoption is self-sustaining.

### The Evolution Pattern

**Problem**: TDS becomes rigid and stops adapting.

**Solution**: Regularly evolve TDS based on team needs.

**Evolution examples**:

-   Adjust comment types for your domain
-   Modify anchor template for your context
-   Create team-specific patterns

**Why it works**: Adaptation keeps TDS relevant and valuable.

## Common Pitfalls and Solutions

### Week 1 Pitfalls

**Over-commenting**

-   Problem: Adding comments to every line
-   Solution: Comment only surprises and discoveries

**Vague comments**

-   Problem: "TODO: Fix this"
-   Solution: "TODO: Handle null user_id from auth service"

**Process focus**

-   Problem: Worrying about perfect categories
-   Solution: Focus on capturing valuable context

### Month 1 Pitfalls

**Over-anchoring**

-   Problem: Creating anchors for trivial things
-   Solution: 30-minute rule - promote if it cost >30 minutes

**Stale promotion**

-   Problem: LEARNINGs sitting in comments for weeks
-   Solution: Weekly review cycle

**Broken references**

-   Problem: Comments referencing non-existent anchors
-   Solution: Link validation in cleanup scripts

### Month 3 Pitfalls

**Pattern paralysis**

-   Problem: Creating patterns too early
-   Solution: Wait for 3+ proven uses

**Context hoarding**

-   Problem: Not sharing knowledge with team
-   Solution: Team TDS review meetings

**Tool complexity**

-   Problem: Over-engineering TDS tooling
-   Solution: Start simple, add complexity gradually

### Month 6 Pitfalls

**Maintenance neglect**

-   Problem: TDS artifacts become stale
-   Solution: Regular cleanup cycles

**Process bureaucracy**

-   Problem: TDS becomes overhead instead of value
-   Solution: Focus on outcomes, not process

**Scale problems**

-   Problem: TDS doesn't scale with team growth
-   Solution: Distribute TDS leadership

## Language-Specific Patterns

### Go-Specific Patterns

```go
// LEARNING: context.WithTimeout doesn't cancel running operations
// Cost: 2 hours debugging why goroutines weren't stopping
// Solution: Pass context to all blocking operations

// Pattern: Always check context in loops
for {
    select {
    case <-ctx.Done():
        return ctx.Err()
    case work := <-workCh:
        // Process work
    }
}
```

### JavaScript-Specific Patterns

```javascript
// LEARNING: Promise.all fails fast on first rejection
// Cost: 3 hours debugging batch processing failures
// Solution: Use Promise.allSettled for partial success handling

// Pattern: Graceful degradation for batch operations
const results = await Promise.allSettled(promises);
const successful = results.filter((r) => r.status === 'fulfilled');
const failed = results.filter((r) => r.status === 'rejected');
```

### Python-Specific Patterns

```python
# LEARNING: requests.Session reuses connections but not timeouts
# Cost: 4 hours debugging hanging requests
# Solution: Set timeout on every request, not just session

# Pattern: Always specify timeouts
session = requests.Session()
response = session.get(url, timeout=30)  # Per-request timeout
```

## Integration Patterns

### API Integration Patterns

```go
// Pattern: Defensive API client
type APIClient struct {
    baseURL     string
    timeout     time.Duration
    retryCount  int
    rateLimiter *rate.Limiter
}

// LEARNING: Always include request context, timeout, and retry logic
// Cost: Multiple production outages from flaky external services
```

### Database Patterns

```sql
-- LEARNING: LIMIT without ORDER BY gives non-deterministic results
-- Cost: 6 hours debugging inconsistent pagination
-- Solution: Always include ORDER BY with LIMIT

-- Pattern: Deterministic pagination
SELECT * FROM users
ORDER BY created_at DESC, id DESC  -- Secondary sort for ties
LIMIT 20 OFFSET 0;
```

---

**Remember**: Best practices evolve with your team. Start with these patterns and adapt based on your specific context and learnings.
