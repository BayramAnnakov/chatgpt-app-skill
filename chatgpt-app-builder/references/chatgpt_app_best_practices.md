# ChatGPT App Best Practices

Guidelines for building high-quality ChatGPT Apps.

## Tool Naming

### Convention
Use `service_verb_noun` pattern in snake_case:
```
taskflow_get_tasks       ✓
taskflow_create_task     ✓
taskflow_complete_task   ✓
getTasks                 ✗ (no prefix, wrong case)
task-list               ✗ (not action-oriented)
```

### Prefix Requirement
Always prefix with service name to prevent conflicts when multiple connectors are active:
```
taskflow_get_tasks
taskflow_create_task
slack_send_message
slack_list_channels
```

### Action Verbs
Use clear, action-oriented verbs:
| Verb | Use For |
|------|---------|
| get | Retrieve single item |
| list | Retrieve collection |
| search | Query with filters |
| create | Make new item |
| update | Modify existing |
| delete | Remove item |
| complete | Mark as done |
| send | Transmit message |

---

## Tool Descriptions

Descriptions are the primary trigger mechanism. Write them carefully.

### Format
Start with "Use this when..." to guide model selection:
```
✓ "Use this when the user wants to see their current tasks,
   optionally filtered by status or due date."

✗ "Gets tasks from the API"  (too vague)
✗ "TaskFlow task retrieval endpoint"  (too technical)
```

### Include
- When to use this tool
- What parameters are available
- What the user will get back

### Avoid
- Marketing language ("Best task manager ever!")
- Recommending overly broad triggering
- Technical implementation details
- Comparisons to competitors

---

## Tool Annotations

Mark tools correctly to inform the model about behavior:

### readOnlyHint: true
For tools that only retrieve data without side effects:
```typescript
// Examples: get_*, list_*, search_*, view_*
_meta: { "openai/readOnlyHint": true }
```

### destructiveHint: true
For tools that delete or irreversibly modify data:
```typescript
// Examples: delete_*, remove_*, archive_*
_meta: { "openai/destructiveHint": true }
```

### openWorldHint: true
For tools that interact with external systems or publish content:
```typescript
// Examples: send_*, publish_*, create_* (when creates external records)
_meta: { "openai/openWorldHint": true }
```

### Combinations
Tools can have multiple hints:
```typescript
// delete_task: removes data (destructive) and affects external system (openWorld)
_meta: {
  "openai/destructiveHint": true,
  "openai/openWorldHint": true
}
```

**Common rejection reason**: Incorrect or missing annotations.

---

## Response Structure

ChatGPT Apps use three response layers:

### content (Model-visible narration)
Text the model sees and can reference in conversation:
```typescript
content: [{
  type: "text",
  text: "Found 5 tasks, 2 are overdue"
}]
```
Keep concise. Model uses this to understand what happened.

### structuredContent (Model-readable data)
Machine-readable data the model can process and reference:
```typescript
structuredContent: {
  tasks: [
    { id: "t1", title: "Review PR", status: "active", due: "2024-01-15" },
    { id: "t2", title: "Deploy", status: "completed" }
  ],
  total: 5,
  overdue: 2
}
```
Include IDs, timestamps, and status fields. Model may use these in follow-up tool calls.

### _meta (Widget-only data)
Rich data only the widget sees, hidden from model:
```typescript
_meta: {
  fullTaskObjects: [...],  // Complete data with all fields
  userPreferences: {...},  // UI preferences
  pagination: { hasMore: true, nextCursor: "abc" }
}
```
Use for large datasets, sensitive data, or widget-specific info.

### Separation Principle
- `structuredContent`: Minimal, what model needs
- `_meta`: Everything widget needs beyond that

---

## Widget State Management

### Persist State Correctly
Use `setWidgetState` for data that should survive widget re-renders:
```typescript
// Good: User selections, filters, UI state
window.openai.setWidgetState({
  selectedTaskId: "t1",
  viewMode: "list",
  sortOrder: "due_date"
});
```

### State Size Limits
Keep state under ~4k tokens. Model receives widget state, so large states waste context.

### State Scope
- State persists within widget instance
- State clears when user types in main composer (starts fresh flow)
- State persists when user interacts via widget controls

---

## Error Handling

Return actionable error messages:

### Good Error Messages
```typescript
// Specific, actionable
"Task not found. The task may have been deleted. Try listing your tasks first."
"Rate limit exceeded. Please wait 60 seconds before trying again."
"Permission denied. You don't have access to this project."
```

### Bad Error Messages
```typescript
// Vague, unhelpful
"Error occurred"
"Request failed"
"500 Internal Server Error"
```

### Error Structure
```typescript
return {
  content: [{
    type: "text",
    text: "Could not complete the action: [specific reason]"
  }],
  structuredContent: {
    error: true,
    errorType: "not_found",
    message: "Task with ID 't999' not found"
  }
};
```

---

## CSP Configuration

Configure Content Security Policy for widget resources:

```typescript
_meta: {
  "openai/widgetCSP": {
    // Domains for fetch/XHR requests
    connect_domains: ["https://api.yourservice.com"],

    // Domains for images, fonts, scripts
    resource_domains: ["https://cdn.yourservice.com"],

    // Domains for openExternal() navigation
    redirect_domains: ["https://checkout.yourservice.com"],

    // Domains for embedded iframes (triggers stricter review)
    frame_domains: ["https://embed.yourservice.com"]
  }
}
```

### Principles
- Only whitelist domains you actually need
- `frame_domains` triggers additional review scrutiny
- Use specific domains, not wildcards when possible

---

## Input Validation

Use Zod with `.strict()` for runtime validation:

```typescript
const GetTasksSchema = z.object({
  status: z.enum(["active", "completed", "all"]).default("all")
    .describe("Filter tasks by status"),
  limit: z.number().int().min(1).max(100).default(20)
    .describe("Maximum number of tasks to return"),
  cursor: z.string().optional()
    .describe("Pagination cursor from previous response")
}).strict();
```

### Requirements
- Use `.describe()` for every parameter
- Add meaningful validation constraints
- Apply `.strict()` to reject unexpected fields
- Set sensible defaults for optional parameters

---

## Design Principles Summary

1. **One job per tool** - Don't combine read + write in one tool
2. **Clear naming** - `service_verb_noun` pattern
3. **Accurate annotations** - Mark read-only, destructive, open-world correctly
4. **Layer separation** - structuredContent for model, _meta for widget
5. **Actionable errors** - Tell users what went wrong and how to fix it
6. **Minimal input** - Only request what's needed for the task
7. **Consistent output** - Same structure across similar tools
