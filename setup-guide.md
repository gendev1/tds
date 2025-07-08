# TDS Setup Guide

This guide will help you implement the Temporal Documentation System in your project, starting simple and building up over time.

## Quick Setup (5 minutes)

### 1. Create Directory Structure

```bash
mkdir -p anchors/{domain,integrations,lessons-learned}
mkdir -p patterns/{error-handling,integrations,testing}
mkdir -p scripts
```

### 2. Create Basic Templates

**anchors/README.md**

```markdown
# Anchors - Institutional Knowledge

## Organization

-   `domain/` - Business domain knowledge
-   `integrations/` - External service learnings
-   `lessons-learned/` - General discoveries

## Template

Each anchor should include:

-   Discovery date
-   Context/situation
-   Key learning
-   Impact/cost if ignored
```

**patterns/README.md**

```markdown
# Patterns - Protective Code

## Organization

-   `error-handling/` - Error handling patterns
-   `integrations/` - Integration safety patterns
-   `testing/` - Testing patterns

## Template

Each pattern should include:

-   Code example
-   When to use
-   What it prevents
-   How to implement
```

### 3. Start Using Comments

Begin adding meaningful comments to your code:

```go
// TODO: API returning 503s intermittently today
// HACK: Working around pagination bug in account service
// NOTE: This timeout might need adjustment during high load
// LEARNING: External API expects camelCase, not snake_case
```

## Week 1: Present Layer Only

Focus on adding meaningful comments as you work:

### Comment Types

```go
// TODO: Investigate why this times out
// HACK: Using 30s timeout for external API
// NOTE: Remember to update this when API changes
// LEARNING: API rate limits at 100 req/min
// FIXME: This breaks with empty responses
```

### Best Practices

-   **Be specific**: "TODO: Fix bug" vs "TODO: Handle null response from user service"
-   **Include context**: "HACK: 30s timeout because vendor API is slow"
-   **Note discoveries**: "LEARNING: API requires UTC timestamps"
-   **Date time-sensitive items**: "NOTE: Temporary fix for Q4 rush (Dec 2024)"

## Week 2-4: Start Anchoring

### Weekly Review Process

1. **Collect learnings**: Find all LEARNING comments from the week
2. **Evaluate significance**: Did this cost time/cause bugs?
3. **Create anchors**: Move valuable learnings to structured documents

### Creating Your First Anchor

```bash
# Example: External API integration learning
touch anchors/integrations/payment-api-timezone.md
```

````markdown
# Payment API Timezone Requirements

**Discovered**: 2024-11-20  
**Context**: Integration with vendor payment system  
**Cost**: 3 hours debugging + failed transactions

## The Issue

Payment API requires all timestamps in EST timezone.
UTC timestamps are silently rejected with 400 error.
No timezone specified = assumes local server time (dangerous).

## Solution

```go
func ToPaymentAPITimestamp(t time.Time) string {
    est, _ := time.LoadLocation("America/New_York")
    return t.In(est).Format(time.RFC3339)
}
```
````

## Impact

-   Failed transactions during timezone confusion
-   Support tickets from confused users
-   3 hours of debugging time

````

### Promotion Guidelines

**Promote to anchors when:**
- Learning cost you >30 minutes
- Mistake could happen again
- Knowledge would help teammates
- External service quirks discovered

**Don't promote:**
- Obvious/trivial discoveries
- One-time fixes
- Personal preferences
- Framework basics

## Month 2: Extract Patterns

### Pattern Identification

Look for repeated solutions in your anchors:

```bash
# Find common themes
grep -r "timezone\|timestamp" anchors/
grep -r "rate limit\|timeout" anchors/
````

### Creating Your First Pattern

```bash
touch patterns/integrations/timezone-safety.md
```

````markdown
# Timezone Safety Pattern

## Problem

External APIs often have undocumented timezone requirements.
Wrong timezone = rejected requests or data corruption.

## Pattern

```go
// Always normalize timezones for external APIs
func ToExternalAPITime(t time.Time, apiTimezone string) string {
    loc, err := time.LoadLocation(apiTimezone)
    if err != nil {
        // Log error but don't fail - use UTC as fallback
        log.Warn("Unknown timezone %s, using UTC", apiTimezone)
        loc = time.UTC
    }
    return t.In(loc).Format(time.RFC3339)
}
```
````

## Usage

```go
// Payment API
paymentTime := ToExternalAPITime(now, "America/New_York")

// Shipping API
shippingTime := ToExternalAPITime(now, "UTC")
```

## Prevents

-   Silent data corruption
-   Debugging timezone issues
-   API rejection errors
-   Cross-timezone bugs

````

## Advanced Setup

### AI Integration

**GitHub Copilot Integration**
```bash
# Create .github/copilot-instructions.md
mkdir -p .github
````

```markdown
# Copilot Instructions

## TDS Integration

This codebase uses Temporal Documentation System:

-   Check /anchors/ for domain knowledge
-   Follow /patterns/ for proven solutions
-   Add LEARNING comments for discoveries

## Code Generation Guidelines

1. Reference relevant anchors in comments
2. Apply patterns for known problems
3. Include error handling from patterns/error-handling/
4. Add TODO comments for incomplete implementations
```

**Claude Integration**

```bash
# Create .claude/project-context.md
mkdir -p .claude
```

```markdown
# Project Context

## TDS System

We use Temporal Documentation System for knowledge management:

-   Present: Comments in code (TODO/HACK/NOTE/LEARNING)
-   Past: /anchors/ directory with lessons learned
-   Future: /patterns/ directory with safeguards

## Before Making Changes

1. Search anchors for relevant knowledge
2. Check patterns for existing solutions
3. Add LEARNING comments for discoveries
```

### Automation Scripts

**TDS Report Script**

```bash
#!/bin/bash
# scripts/tds-report.sh

echo "=== TDS Weekly Report ==="
echo ""
echo "Active Learnings:"
grep -r "LEARNING:" --include="*.go" . | grep -v "/vendor/"
echo ""
echo "Recent Anchors (last 7 days):"
find ./anchors -name "*.md" -mtime -7
echo ""
echo "TODOs needing attention:"
grep -r "TODO:" --include="*.go" . | grep -v "/vendor/" | head -10
```

**Learning Promotion Helper**

```bash
#!/bin/bash
# scripts/promote-learnings.sh

# Find all LEARNING comments
echo "Found LEARNING comments:"
grep -rn "LEARNING:" --include="*.go" . | grep -v "/vendor/"

echo ""
echo "Create anchors for valuable learnings:"
echo "1. Choose significant learnings (>30min impact)"
echo "2. Create anchor file: anchors/category/topic.md"
echo "3. Include: date, context, learning, impact"
echo "4. Remove or update the original comment"
```

### Directory Organization

```
project/
├── .github/
│   └── copilot-instructions.md
├── .claude/
│   └── project-context.md
├── anchors/
│   ├── domain/
│   │   ├── user-permissions.md
│   │   └── business-rules.md
│   ├── integrations/
│   │   ├── payment-api.md
│   │   └── shipping-service.md
│   ├── lessons-learned/
│   │   ├── database-migrations.md
│   │   └── deployment-gotchas.md
│   └── README.md
├── patterns/
│   ├── error-handling/
│   │   ├── retry-logic.md
│   │   └── timeout-handling.md
│   ├── integrations/
│   │   ├── api-client-safety.md
│   │   └── webhook-verification.md
│   ├── testing/
│   │   ├── integration-test-setup.md
│   │   └── mock-strategies.md
│   └── README.md
└── scripts/
    ├── tds-report.sh
    └── promote-learnings.sh
```

## Maturity Progression

### Level 1: Reactive (Months 1-2)

-   Adding LEARNING comments when surprised
-   Basic anchor documents
-   Manual promotion process

### Level 2: Proactive (Months 3-6)

-   Regular weekly reviews
-   Domain-organized anchors
-   First patterns emerging
-   Team adoption

### Level 3: Systematic (Months 6-12)

-   Automated reporting
-   AI integration working
-   Patterns preventing issues
-   Self-organizing knowledge

### Level 4: Cultural (Year 2+)

-   TDS is "how we work"
-   Natural knowledge flow
-   Minimal repeat issues
-   Organizational learning

## Common Pitfalls

### Week 1 Pitfalls

-   **Over-commenting**: Not every line needs a comment
-   **Vague comments**: "TODO: Fix this" vs "TODO: Handle null user ID"
-   **Permanent TODOs**: Set review dates or priorities

### Month 1 Pitfalls

-   **Over-anchoring**: Don't document trivial things
-   **Stale comments**: Remove/update promoted learnings
-   **No promotion**: All learnings stuck in comments

### Month 3 Pitfalls

-   **Pattern paralysis**: Creating patterns before proving value
-   **Context hoarding**: Not sharing learnings with team
-   **Tool complexity**: Over-engineering the system

## Success Indicators

### Week 1

-   [ ] Comments are specific and contextual
-   [ ] LEARNINGs captured during debugging
-   [ ] Team understands comment types

### Month 1

-   [ ] First anchors created
-   [ ] Weekly review process established
-   [ ] Some comments promoted/cleaned

### Month 3

-   [ ] First patterns extracted
-   [ ] Team referencing anchors
-   [ ] Reduced repeat issues

### Month 6

-   [ ] AI integration working
-   [ ] Self-organizing knowledge
-   [ ] Cultural adoption

---

Remember: **Start simple, evolve naturally**. TDS grows with your team's needs.
