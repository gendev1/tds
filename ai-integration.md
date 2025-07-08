# AI Integration with TDS

TDS amplifies AI coding assistants by providing rich context, proven patterns, and institutional memory that ground AI suggestions in reality.

## The AI-TDS Synergy

### What AI Gets from TDS

-   **Rich Context**: Anchors explain the "why" behind code decisions
-   **Proven Patterns**: Established solutions that work in your environment
-   **Failure Prevention**: Knowledge of what doesn't work
-   **Domain Knowledge**: Business logic and constraints specific to your system

### What TDS Gets from AI

-   **Faster Discovery**: AI helps identify potential issues
-   **Pattern Implementation**: AI applies patterns consistently
-   **Documentation Generation**: AI helps create anchor content
-   **Code Generation**: AI creates code following TDS patterns

## GitHub Copilot Integration

### Setup: Copilot Instructions File

Create `.github/copilot-instructions.md`:

````markdown
# GitHub Copilot Instructions

## Temporal Documentation System (TDS)

This codebase uses TDS for knowledge management. Before generating code:

1. **Check Context**: Look for relevant anchors and patterns
2. **Follow Patterns**: Apply established solutions from /patterns/
3. **Reference Anchors**: Include links to relevant institutional knowledge
4. **Add Learnings**: Include LEARNING comments for new discoveries

## TDS Structure

### Present Layer (Comments)

-   TODO: Known issues needing attention
-   HACK: Temporary workarounds with context
-   NOTE: Important context for future changes
-   LEARNING: Fresh discoveries and insights
-   FIXME: Bugs requiring immediate attention

### Past Layer (Anchors)

-   `/anchors/domain/` - Business domain knowledge
-   `/anchors/integrations/` - External service learnings
-   `/anchors/lessons-learned/` - Technical discoveries

### Protection Layer (Patterns)

-   `/patterns/error-handling/` - Error handling patterns
-   `/patterns/integrations/` - Integration safety patterns
-   `/patterns/testing/` - Testing patterns

## Code Generation Guidelines

### Always Include Error Handling

Follow patterns from `/patterns/error-handling/` for consistent error management.

### External API Calls

Use patterns from `/patterns/integrations/api-client-safety.md` for all external service calls.

### Database Operations

Check `/anchors/lessons-learned/database-performance.md` for performance considerations.

### Authentication/Authorization

Reference `/anchors/domain/user-permissions.md` for permission model.

## Comment Style

### Good Comments

```go
// LEARNING: Payment API requires EST timezone (see anchors/integrations/payment-api.md)
// TODO: Add retry logic following patterns/integrations/api-client-safety.md
// HACK: Temporary fix for vendor API bug (remove after Q1 2025 update)
```
````

### Avoid

```go
// This function processes payments (generic)
// TODO: Fix this (vague)
// Works for now (no context)
```

## When Generating Code

1. **Reference relevant anchors** in comments
2. **Apply established patterns** for common scenarios
3. **Include context** about decisions made
4. **Add LEARNING comments** for new discoveries
5. **Follow existing code style** and patterns

````

### Copilot Prompting Strategies

#### Context-Aware Prompting
```go
// Generate payment processor following:
// - Anchors: /anchors/integrations/payment-api.md
// - Patterns: /patterns/integrations/api-client-safety.md
// - Include timezone handling (EST required)
// - Add retry logic for rate limits
````

#### Pattern-Based Prompting

```go
// Implement user permission check using:
// - Pattern: /patterns/security/permission-checking.md
// - Domain model: /anchors/domain/user-permissions.md
// - Handle inheritance: user -> role -> group -> org
```

#### Learning-Oriented Prompting

```go
// Create webhook handler with:
// - LEARNING comments for new discoveries
// - Reference to /anchors/integrations/webhook-security.md
// - Signature verification pattern
```

### Copilot Comment Integration

#### Inline Context References

```go
// Copilot: check anchors/integrations/payment-api.md before generating
func ProcessPayment(req PaymentRequest) error {
    // Implementation follows established patterns
}
```

#### Pattern Application

```go
// Copilot: use pattern from patterns/integrations/api-client-safety.md
client := NewSafeAPIClient(config)
```

#### Learning Capture

```go
// Copilot: add LEARNING comment if this behavior is surprising
result, err := externalAPI.Call(request)
```

## Claude Integration

### Setup: Project Context File

Create `.claude/project-context.md`:

```markdown
# Project Context for Claude

## Temporal Documentation System (TDS)

This project uses TDS for knowledge management across three temporal layers:

### Present Layer (Active Context)

Comments in code capture immediate discoveries and context:

-   TODO: Known issues needing attention
-   HACK: Temporary workarounds with context
-   NOTE: Important context for future changes
-   LEARNING: Fresh discoveries and insights
-   FIXME: Bugs requiring immediate attention

### Past Layer (Institutional Memory)

Anchors preserve lessons learned and domain knowledge:

-   `/anchors/domain/` - Business domain knowledge
-   `/anchors/integrations/` - External service learnings
-   `/anchors/lessons-learned/` - Technical discoveries

### Protection Layer (Proven Patterns)

Patterns provide reusable solutions and safeguards:

-   `/patterns/error-handling/` - Error handling patterns
-   `/patterns/integrations/` - Integration safety patterns
-   `/patterns/testing/` - Testing patterns

## Before Making Changes

1. **Search anchors** for relevant domain knowledge
2. **Check patterns** for existing solutions
3. **Review recent LEARNING comments** for current context
4. **Add LEARNING comments** for new discoveries

## Code Generation Guidelines

### Always Reference Context

Include links to relevant anchors and patterns in code comments.

### Follow Established Patterns

Use proven solutions from `/patterns/` directory.

### Capture New Learnings

Add LEARNING comments for surprising discoveries.

### Maintain TDS Structure

Update anchors and patterns as knowledge evolves.

## Common Patterns to Follow

### API Client Safety

All external API calls should follow `/patterns/integrations/api-client-safety.md`

### Error Handling

Use patterns from `/patterns/error-handling/` for consistent error management.

### Database Operations

Check `/anchors/lessons-learned/database-performance.md` for performance considerations.

### User Permissions

Reference `/anchors/domain/user-permissions.md` for permission model.
```

### Claude Task Templates

#### Investigation Template

```markdown
# Task: Investigate [Issue]

## TDS Context Review

-   [ ] Check anchors for related knowledge
-   [ ] Review patterns for similar solutions
-   [ ] Scan recent LEARNING comments

## Expected TDS Output

-   [ ] LEARNING comments for discoveries
-   [ ] Update relevant anchors if needed
-   [ ] Extract patterns if reusable solutions emerge
```

#### Feature Implementation Template

```markdown
# Task: Implement [Feature]

## TDS Context Needed

-   [ ] Domain knowledge from anchors/domain/
-   [ ] Integration patterns from patterns/integrations/
-   [ ] Relevant lessons from anchors/lessons-learned/

## Implementation Guidelines

-   [ ] Follow established patterns
-   [ ] Reference context in comments
-   [ ] Add LEARNING comments for surprises
-   [ ] Update anchors if significant discoveries
```

### Claude Conversation Starters

#### Context-Rich Requests

```
I need to implement user authentication. Please:
1. Check anchors/domain/user-permissions.md for our permission model
2. Review patterns/security/ for established auth patterns
3. Include LEARNING comments for any surprising discoveries
4. Reference relevant anchors in the implementation
```

#### Pattern-Based Requests

```
Create an API client for the new vendor service following:
- patterns/integrations/api-client-safety.md
- anchors/integrations/vendor-api-quirks.md
- Include timeout/retry logic
- Add LEARNING comments for any API surprises
```

### Claude Code Commands

Create helpful aliases for TDS operations:

```bash
# .zshrc or .bashrc
alias tds-search="grep -r 'LEARNING\|TODO\|HACK' --include='*.go' ."
alias tds-anchors="find anchors/ -name '*.md' | head -20"
alias tds-patterns="find patterns/ -name '*.md' | head -20"
alias tds-learnings="grep -r 'LEARNING:' --include='*.go' . | head -10"
```

## AI-Enhanced TDS Workflows

### Discovery Enhancement

```bash
# AI helps identify potential issues
claude "Review this code for potential issues, check against our TDS anchors"

# AI suggests improvements based on patterns
copilot "Improve this following our established patterns"
```

### Promotion Assistance

```bash
# AI helps create anchor content
claude "Create anchor document from this LEARNING comment"

# AI identifies pattern candidates
claude "Find similar solutions across our anchors that could become patterns"
```

### Pattern Generation

```bash
# AI extracts patterns from multiple anchors
claude "Extract common pattern from these similar anchor documents"

# AI suggests pattern variations
claude "What variations of this pattern might we need?"
```

## AI Integration Best Practices

### Do's

-   **Provide rich context** through anchors and patterns
-   **Reference TDS structure** in AI prompts
-   **Use AI to enhance** TDS workflows, not replace them
-   **Validate AI suggestions** against existing anchors
-   **Capture AI discoveries** in LEARNING comments

### Don'ts

-   **Blindly follow AI suggestions** without TDS context
-   **Skip pattern checking** before implementing AI code
-   **Ignore institutional knowledge** in favor of AI novelty
-   **Let AI create patterns** without proving them first
-   **Forget to update TDS** based on AI interactions

### Context Optimization

#### Anchor References in Prompts

```
Before implementing payment processing:
- Check anchors/integrations/payment-api.md
- Review patterns/integrations/api-client-safety.md
- Consider lessons from anchors/lessons-learned/payment-failures.md
```

#### Pattern Application in Prompts

```
Generate error handling code using:
- patterns/error-handling/retry-logic.md
- patterns/error-handling/circuit-breaker.md
- Specific to our payment service context
```

### Learning Amplification

#### AI-Discovered Learnings

```go
// LEARNING: AI suggested this optimization based on our patterns
// Reduced API calls by 40% using request batching
// (AI identified pattern from anchors/integrations/batch-optimization.md)
```

#### Pattern Evolution Through AI

```go
// AI identified this variation of our retry pattern
// LEARNING: Exponential backoff with jitter works better for our use case
// Consider updating patterns/error-handling/retry-logic.md
```

## Measuring AI-TDS Integration Success

### Metrics to Track

#### Context Utilization

-   **Anchor references** in AI-generated code
-   **Pattern applications** in AI implementations
-   **Learning captures** during AI interactions

#### Quality Improvements

-   **First-time success rate** of AI suggestions
-   **Reduced debugging time** from better context
-   **Pattern consistency** in AI-generated code

#### Knowledge Evolution

-   **AI-discovered patterns** promoted to protection layer
-   **Anchor updates** based on AI interactions
-   **Pattern refinements** from AI usage

### Success Indicators

#### Short-term (1-3 months)

-   AI generates code referencing anchors
-   AI applies established patterns consistently
-   AI discoveries captured in LEARNING comments

#### Medium-term (3-6 months)

-   AI helps identify pattern candidates
-   AI-generated code follows TDS conventions
-   AI assists in anchor content creation

#### Long-term (6+ months)

-   AI-TDS integration feels natural
-   AI amplifies institutional knowledge
-   AI helps evolve TDS structure

---

**Remember**: AI is a powerful amplifier of TDS, not a replacement. The combination of AI's generation capabilities with TDS's institutional memory creates a compound effect that improves over time.
