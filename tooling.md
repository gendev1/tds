# TDS Tooling and Automation

Effective TDS implementation benefits from automation and tooling that makes the knowledge flow feel natural and low-friction.

## Core Scripts

### TDS Report Script

**Purpose**: Weekly overview of TDS system health and actionable items.

**File**: `scripts/tds-report.sh`

```bash
#!/bin/bash
# TDS Weekly Report Generator

echo "======================================="
echo "TDS WEEKLY REPORT - $(date)"
echo "======================================="
echo ""

# Active Learnings
echo "🔍 ACTIVE LEARNINGS"
echo "-------------------"
LEARNINGS=$(grep -r "LEARNING:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/")
if [ -n "$LEARNINGS" ]; then
    echo "$LEARNINGS"
else
    echo "No new learnings found this week."
fi
echo ""

# Recent Anchors
echo "⚓ RECENT ANCHORS (last 7 days)"
echo "------------------------------"
RECENT_ANCHORS=$(find ./anchors -name "*.md" -mtime -7 2>/dev/null)
if [ -n "$RECENT_ANCHORS" ]; then
    echo "$RECENT_ANCHORS"
else
    echo "No new anchors created this week."
fi
echo ""

# TODOs Needing Attention
echo "📋 TODOs NEEDING ATTENTION"
echo "-------------------------"
OLD_TODOS=$(grep -r "TODO:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | head -10)
if [ -n "$OLD_TODOS" ]; then
    echo "$OLD_TODOS"
else
    echo "No active TODOs found."
fi
echo ""

# Stale Comments
echo "🗑️  STALE COMMENTS (>30 days)"
echo "-----------------------------"
STALE_COMMENTS=$(grep -r "HACK:\|FIXME:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | grep -E "(Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec) 202[0-3]")
if [ -n "$STALE_COMMENTS" ]; then
    echo "$STALE_COMMENTS"
else
    echo "No stale comments found."
fi
echo ""

# Pattern Candidates
echo "🔧 PATTERN CANDIDATES"
echo "--------------------"
echo "Searching for repeated solutions..."
PATTERN_CANDIDATES=$(grep -r "Similar to\|Like in\|Same as" anchors/ 2>/dev/null | head -5)
if [ -n "$PATTERN_CANDIDATES" ]; then
    echo "$PATTERN_CANDIDATES"
else
    echo "No obvious pattern candidates found."
fi
echo ""

# TDS Stats
echo "📊 TDS STATISTICS"
echo "----------------"
ANCHOR_COUNT=$(find ./anchors -name "*.md" 2>/dev/null | wc -l)
PATTERN_COUNT=$(find ./patterns -name "*.md" 2>/dev/null | wc -l)
LEARNING_COUNT=$(grep -r "LEARNING:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | wc -l)
TODO_COUNT=$(grep -r "TODO:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | wc -l)

echo "Anchors: $ANCHOR_COUNT"
echo "Patterns: $PATTERN_COUNT"
echo "Active LEARNINGs: $LEARNING_COUNT"
echo "Active TODOs: $TODO_COUNT"
echo ""

echo "======================================="
echo "Run 'scripts/promote-learnings.sh' to promote valuable learnings"
echo "Run 'scripts/tds-cleanup.sh' to clean up stale comments"
echo "======================================="
```

### Learning Promotion Helper

**Purpose**: Assists in promoting LEARNING comments to anchors.

**File**: `scripts/promote-learnings.sh`

```bash
#!/bin/bash
# Learning Promotion Helper

echo "======================================="
echo "TDS LEARNING PROMOTION HELPER"
echo "======================================="
echo ""

# Find all LEARNING comments
echo "🔍 FOUND LEARNING COMMENTS:"
echo "----------------------------"
LEARNINGS=$(grep -rn "LEARNING:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/")

if [ -z "$LEARNINGS" ]; then
    echo "No LEARNING comments found."
    exit 0
fi

echo "$LEARNINGS"
echo ""

# Identify high-value learnings
echo "💡 HIGH-VALUE LEARNINGS (potential candidates):"
echo "----------------------------------------------"
HIGH_VALUE=$(echo "$LEARNINGS" | grep -E "(hours|cost|debug|failed|broke|timeout|error|bug|issue)")
if [ -n "$HIGH_VALUE" ]; then
    echo "$HIGH_VALUE"
else
    echo "No high-value learnings detected automatically."
fi
echo ""

# Promotion workflow
echo "📝 PROMOTION WORKFLOW:"
echo "----------------------"
echo "1. Review learnings above for promotion value"
echo "2. Create anchor file: mkdir -p anchors/[category] && touch anchors/[category]/[topic].md"
echo "3. Use template from anchors/README.md"
echo "4. Include: discovery date, context, cost, learning, impact"
echo "5. Update/remove original LEARNING comment"
echo "6. Reference anchor in code if needed"
echo ""

# Anchor categories
echo "📁 AVAILABLE ANCHOR CATEGORIES:"
echo "-------------------------------"
if [ -d "./anchors" ]; then
    find ./anchors -type d -not -path "./anchors" | sed 's|./anchors/||' | sort
else
    echo "domain/ - Business domain knowledge"
    echo "integrations/ - External service learnings"
    echo "lessons-learned/ - Technical discoveries"
fi
echo ""

# Template reminder
echo "📋 ANCHOR TEMPLATE REMINDER:"
echo "----------------------------"
echo "# [Topic] - [Brief Description]"
echo ""
echo "**Discovered**: $(date +%Y-%m-%d)"
echo "**Context**: [Situation/circumstances]"
echo "**Cost**: [Time/impact if not known]"
echo ""
echo "## The Issue"
echo "[Detailed description]"
echo ""
echo "## The Solution"
echo "[What was learned]"
echo ""
echo "## Impact"
echo "- [Consequences of not knowing]"
echo "- [Benefits of following approach]"
echo ""
echo "======================================="
```

### TDS Cleanup Script

**Purpose**: Helps maintain clean, current comment state.

**File**: `scripts/tds-cleanup.sh`

```bash
#!/bin/bash
# TDS Cleanup Helper

echo "======================================="
echo "TDS CLEANUP HELPER"
echo "======================================="
echo ""

# Find stale TODOs
echo "🗑️  STALE TODOs (>90 days old):"
echo "-------------------------------"
STALE_TODOS=$(grep -rn "TODO:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | grep -E "(Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec) 202[0-2]")
if [ -n "$STALE_TODOS" ]; then
    echo "$STALE_TODOS"
else
    echo "No stale TODOs found."
fi
echo ""

# Find completed HACKs
echo "🔧 COMPLETED HACKs (may be removable):"
echo "-------------------------------------"
COMPLETED_HACKS=$(grep -rn "HACK:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | grep -E "(fixed|resolved|completed|done)")
if [ -n "$COMPLETED_HACKS" ]; then
    echo "$COMPLETED_HACKS"
else
    echo "No completed HACKs found."
fi
echo ""

# Find promoted LEARNINGs
echo "📚 PROMOTED LEARNINGs (may need updating):"
echo "-----------------------------------------"
PROMOTED_LEARNINGS=$(grep -rn "LEARNING:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | grep -E "(anchor|pattern)")
if [ -n "$PROMOTED_LEARNINGS" ]; then
    echo "$PROMOTED_LEARNINGS"
else
    echo "No promoted LEARNINGs found."
fi
echo ""

# Broken anchor references
echo "🔗 BROKEN ANCHOR REFERENCES:"
echo "----------------------------"
ANCHOR_REFS=$(grep -rn "anchors/" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | grep -E "//.*anchors/")
if [ -n "$ANCHOR_REFS" ]; then
    echo "Checking references..."
    while IFS= read -r line; do
        FILE=$(echo "$line" | cut -d: -f1)
        REF=$(echo "$line" | grep -o 'anchors/[^)]*\.md' | head -1)
        if [ -n "$REF" ] && [ ! -f "$REF" ]; then
            echo "BROKEN: $FILE references missing $REF"
        fi
    done <<< "$ANCHOR_REFS"
else
    echo "No anchor references found in comments."
fi
echo ""

# Pattern references
echo "🔧 PATTERN REFERENCES:"
echo "----------------------"
PATTERN_REFS=$(grep -rn "patterns/" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | grep -E "//.*patterns/")
if [ -n "$PATTERN_REFS" ]; then
    echo "Checking references..."
    while IFS= read -r line; do
        FILE=$(echo "$line" | cut -d: -f1)
        REF=$(echo "$line" | grep -o 'patterns/[^)]*\.md' | head -1)
        if [ -n "$REF" ] && [ ! -f "$REF" ]; then
            echo "BROKEN: $FILE references missing $REF"
        fi
    done <<< "$PATTERN_REFS"
else
    echo "No pattern references found in comments."
fi
echo ""

echo "======================================="
echo "CLEANUP RECOMMENDATIONS:"
echo "1. Review and resolve/remove stale TODOs"
echo "2. Clean up completed HACKs"
echo "3. Update promoted LEARNINGs to reference anchors"
echo "4. Fix broken anchor/pattern references"
echo "======================================="
```

## Pattern Search Tools

### Pattern Identification Script

**Purpose**: Identifies potential patterns from anchor similarities.

**File**: `scripts/find-patterns.sh`

````bash
#!/bin/bash
# Pattern Identification Helper

echo "======================================="
echo "TDS PATTERN IDENTIFICATION"
echo "======================================="
echo ""

if [ ! -d "./anchors" ]; then
    echo "No anchors directory found. Create some anchors first!"
    exit 1
fi

# Find common terms across anchors
echo "🔍 COMMON THEMES IN ANCHORS:"
echo "----------------------------"
echo "Top terms appearing in multiple anchors:"
cat anchors/**/*.md 2>/dev/null | grep -o '\b[a-z]\{4,\}\b' | sort | uniq -c | sort -nr | head -20
echo ""

# Find similar anchor titles
echo "📝 SIMILAR ANCHOR TITLES:"
echo "-------------------------"
find ./anchors -name "*.md" -exec basename {} \; | sort | uniq -c | sort -nr | awk '$1 > 1'
echo ""

# Find repeated solutions
echo "🔧 REPEATED SOLUTIONS:"
echo "---------------------"
echo "Searching for similar code patterns in anchors..."
grep -r "```" anchors/ | grep -E "(timeout|retry|client|error|auth)" | head -10
echo ""

# Find integration patterns
echo "🔗 INTEGRATION PATTERNS:"
echo "------------------------"
echo "External service related anchors:"
find ./anchors -name "*.md" -exec grep -l "API\|service\|external\|integration" {} \; | head -10
echo ""

# Find error handling patterns
echo "❌ ERROR HANDLING PATTERNS:"
echo "---------------------------"
echo "Error related anchors:"
find ./anchors -name "*.md" -exec grep -l "error\|timeout\|retry\|fail" {} \; | head -10
echo ""

echo "======================================="
echo "PATTERN CREATION WORKFLOW:"
echo "1. Review similar anchors above"
echo "2. Extract common solution"
echo "3. Create pattern in patterns/[category]/[name].md"
echo "4. Update anchors to reference pattern"
echo "5. Use pattern in new code"
echo "======================================="
````

### TDS Search Tool

**Purpose**: Comprehensive search across all TDS layers.

**File**: `scripts/tds-search.sh`

```bash
#!/bin/bash
# TDS Search Tool

SEARCH_TERM="$1"

if [ -z "$SEARCH_TERM" ]; then
    echo "Usage: $0 <search_term>"
    echo "Example: $0 'payment api'"
    exit 1
fi

echo "======================================="
echo "TDS SEARCH RESULTS for: $SEARCH_TERM"
echo "======================================="
echo ""

# Search in present layer (comments)
echo "💬 PRESENT LAYER (Comments):"
echo "----------------------------"
COMMENT_RESULTS=$(grep -rn "$SEARCH_TERM" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | grep -E "//.*($SEARCH_TERM)")
if [ -n "$COMMENT_RESULTS" ]; then
    echo "$COMMENT_RESULTS"
else
    echo "No matches in comments."
fi
echo ""

# Search in past layer (anchors)
echo "⚓ PAST LAYER (Anchors):"
echo "-----------------------"
if [ -d "./anchors" ]; then
    ANCHOR_RESULTS=$(grep -rn "$SEARCH_TERM" anchors/)
    if [ -n "$ANCHOR_RESULTS" ]; then
        echo "$ANCHOR_RESULTS"
    else
        echo "No matches in anchors."
    fi
else
    echo "No anchors directory found."
fi
echo ""

# Search in protection layer (patterns)
echo "🛡️  PROTECTION LAYER (Patterns):"
echo "--------------------------------"
if [ -d "./patterns" ]; then
    PATTERN_RESULTS=$(grep -rn "$SEARCH_TERM" patterns/)
    if [ -n "$PATTERN_RESULTS" ]; then
        echo "$PATTERN_RESULTS"
    else
        echo "No matches in patterns."
    fi
else
    echo "No patterns directory found."
fi
echo ""

# Search in code implementation
echo "💻 CODE IMPLEMENTATION:"
echo "----------------------"
CODE_RESULTS=$(grep -rn "$SEARCH_TERM" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | grep -v -E "//.*($SEARCH_TERM)" | head -10)
if [ -n "$CODE_RESULTS" ]; then
    echo "$CODE_RESULTS"
else
    echo "No matches in code."
fi
echo ""

echo "======================================="
```

## Development Integration

### Git Hooks

**Purpose**: Automatically run TDS checks during development.

**File**: `.git/hooks/pre-commit`

```bash
#!/bin/bash
# TDS Pre-commit Hook

echo "Running TDS checks..."

# Check for LEARNING comments without context
POOR_LEARNINGS=$(git diff --cached --name-only | xargs grep -l "LEARNING:" 2>/dev/null | xargs grep "LEARNING:" | grep -v -E "(cost|time|debug|found|discovered)")
if [ -n "$POOR_LEARNINGS" ]; then
    echo "Warning: LEARNING comments found without context:"
    echo "$POOR_LEARNINGS"
    echo "Consider adding context about time cost or impact."
fi

# Check for vague TODOs
VAGUE_TODOS=$(git diff --cached --name-only | xargs grep -l "TODO:" 2>/dev/null | xargs grep "TODO:" | grep -E "(fix this|update this|change this)")
if [ -n "$VAGUE_TODOS" ]; then
    echo "Warning: Vague TODO comments found:"
    echo "$VAGUE_TODOS"
    echo "Consider adding specific context about what needs to be done."
fi

# Check for broken anchor references
BROKEN_REFS=$(git diff --cached --name-only | xargs grep -l "anchors/" 2>/dev/null | xargs grep "anchors/" | grep -o 'anchors/[^)]*\.md' | while read ref; do
    if [ ! -f "$ref" ]; then
        echo "Broken reference: $ref"
    fi
done)
if [ -n "$BROKEN_REFS" ]; then
    echo "Error: Broken anchor references found:"
    echo "$BROKEN_REFS"
    echo "Please fix references before committing."
    exit 1
fi

echo "TDS checks passed."
```

### VS Code Integration

**Purpose**: TDS support in VS Code.

**File**: `.vscode/settings.json`

```json
{
    "todo-tree.regex.regex": "((//|#|<!--)\\s*($TAGS)\\s*:?\\s*(.*))",
    "todo-tree.general.tags": ["TODO", "HACK", "NOTE", "LEARNING", "FIXME"],
    "todo-tree.highlights.customHighlight": {
        "TODO": {
            "icon": "check",
            "iconColour": "#FFD700"
        },
        "HACK": {
            "icon": "tools",
            "iconColour": "#FFA500"
        },
        "NOTE": {
            "icon": "note",
            "iconColour": "#00BFFF"
        },
        "LEARNING": {
            "icon": "light-bulb",
            "iconColour": "#90EE90"
        },
        "FIXME": {
            "icon": "alert",
            "iconColour": "#FF6347"
        }
    },
    "files.associations": {
        "anchors/**/*.md": "markdown",
        "patterns/**/*.md": "markdown"
    }
}
```

**File**: `.vscode/tasks.json`

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "TDS Report",
            "type": "shell",
            "command": "${workspaceFolder}/scripts/tds-report.sh",
            "group": "build",
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared"
            },
            "problemMatcher": []
        },
        {
            "label": "TDS Promote Learnings",
            "type": "shell",
            "command": "${workspaceFolder}/scripts/promote-learnings.sh",
            "group": "build",
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared"
            },
            "problemMatcher": []
        },
        {
            "label": "TDS Cleanup",
            "type": "shell",
            "command": "${workspaceFolder}/scripts/tds-cleanup.sh",
            "group": "build",
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared"
            },
            "problemMatcher": []
        },
        {
            "label": "TDS Search",
            "type": "shell",
            "command": "${workspaceFolder}/scripts/tds-search.sh",
            "group": "build",
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared"
            },
            "problemMatcher": []
        }
    ]
}
```

### Shell Aliases

**Purpose**: Quick access to TDS operations.

**File**: Add to `.bashrc` or `.zshrc`

```bash
# TDS Aliases
alias tds-report='./scripts/tds-report.sh'
alias tds-promote='./scripts/promote-learnings.sh'
alias tds-cleanup='./scripts/tds-cleanup.sh'
alias tds-search='./scripts/tds-search.sh'
alias tds-patterns='./scripts/find-patterns.sh'

# TDS Search Shortcuts
alias tds-learnings='grep -r "LEARNING:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/"'
alias tds-todos='grep -r "TODO:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/"'
alias tds-hacks='grep -r "HACK:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/"'

# TDS Navigation
alias tds-anchors='ls -la anchors/'
alias tds-patterns-ls='ls -la patterns/'
alias tds-recent='find ./anchors -name "*.md" -mtime -7'
```

## Automation Scripts

### Weekly TDS Review Automation

**Purpose**: Automated weekly TDS health check.

**File**: `scripts/weekly-tds-review.sh`

```bash
#!/bin/bash
# Weekly TDS Review Automation

# Run on Friday afternoons
# Add to cron: 0 16 * * 5 /path/to/scripts/weekly-tds-review.sh

REPORT_FILE="tds-weekly-report-$(date +%Y%m%d).md"

echo "# TDS Weekly Review - $(date +%Y-%m-%d)" > "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

# Generate standard report
./scripts/tds-report.sh >> "$REPORT_FILE"

# Add team discussion prompts
echo "" >> "$REPORT_FILE"
echo "## Team Discussion Items" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"
echo "- [ ] Which learnings should be promoted to anchors?" >> "$REPORT_FILE"
echo "- [ ] Are there patterns emerging from recent anchors?" >> "$REPORT_FILE"
echo "- [ ] What TODOs can be resolved or reprioritized?" >> "$REPORT_FILE"
echo "- [ ] Any stale comments that should be cleaned up?" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

# Email or post to team channel
if command -v mail &> /dev/null; then
    mail -s "TDS Weekly Review - $(date +%Y-%m-%d)" team@company.com < "$REPORT_FILE"
fi

echo "Weekly TDS review generated: $REPORT_FILE"
```

### TDS Metrics Collection

**Purpose**: Collect metrics for TDS effectiveness.

**File**: `scripts/tds-metrics.sh`

```bash
#!/bin/bash
# TDS Metrics Collection

METRICS_FILE="tds-metrics-$(date +%Y%m%d).json"

# Count various TDS elements
ANCHOR_COUNT=$(find ./anchors -name "*.md" 2>/dev/null | wc -l)
PATTERN_COUNT=$(find ./patterns -name "*.md" 2>/dev/null | wc -l)
LEARNING_COUNT=$(grep -r "LEARNING:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | wc -l)
TODO_COUNT=$(grep -r "TODO:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | wc -l)
HACK_COUNT=$(grep -r "HACK:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | wc -l)

# Generate JSON metrics
cat > "$METRICS_FILE" << EOF
{
  "date": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "anchors": {
    "total": $ANCHOR_COUNT,
    "recent": $(find ./anchors -name "*.md" -mtime -7 2>/dev/null | wc -l)
  },
  "patterns": {
    "total": $PATTERN_COUNT,
    "recent": $(find ./patterns -name "*.md" -mtime -30 2>/dev/null | wc -l)
  },
  "comments": {
    "learnings": $LEARNING_COUNT,
    "todos": $TODO_COUNT,
    "hacks": $HACK_COUNT
  },
  "health": {
    "stale_todos": $(grep -r "TODO:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | grep -E "(Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec) 202[0-2]" | wc -l),
    "high_value_learnings": $(grep -r "LEARNING:" --include="*.go" --include="*.js" --include="*.py" . | grep -v "/vendor/" | grep -v "/node_modules/" | grep -E "(hours|cost|debug|failed)" | wc -l)
  }
}
EOF

echo "TDS metrics collected: $METRICS_FILE"
```

---

**Remember**: Tools should support the TDS workflow, not complicate it. Start with the basic scripts and add automation as your team's TDS maturity grows.
