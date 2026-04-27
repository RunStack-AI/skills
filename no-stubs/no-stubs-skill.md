# Cursor Rule: No Stubs, No Fake Implementations

## Core Directive

**Never write stub implementations.** Every function, endpoint, service, and integration you write must be fully implemented and functional. If something cannot be completed right now, stop and ask — do not fake it.

---

## What "Stubbing" Means (and Why It's Banned)

A stub is any code that pretends to work but doesn't. This includes:

- Functions that return hardcoded or fake values (e.g. `return {"status": "ok"}`)
- Async functions that just `await asyncio.sleep(0.1)` and return fake output
- Routes or handlers that exist but do nothing
- Logic that is defined but never wired up or called
- Placeholder comments like `# TODO: implement this` inside running code paths
- Safety, auth, or validation layers that always pass without checking anything

These are **silent failures**. The system appears to work during development but crashes or produces garbage in production.

---

## Rules

### 1. If you can't implement it now, don't create it

Do not create a function, class, route, or service unless you are implementing it fully in the same session. A stub is not better than nothing — it is worse, because it hides the gap.

**Wrong:**
```python
async def send_notification(user_id: str, message: str):
    # TODO: wire up to email service
    pass
```

**Right:** Either implement it, or don't create the function yet. Ask the developer what service to use and implement it properly.

---

### 2. Never import a stub in production code paths

If a stub file exists for development or testing purposes, it must never be imported in any live code path. Stub files belong only in test fixtures.

**Wrong:**
```python
# In orchestrator.py
from stubs.task_stub import run_task  # used in production flow
```

**Right:** Implement `run_task` using the actual service it's supposed to call.

---

### 3. Implement the real thing, even if it's harder

When a full implementation already exists somewhere in the codebase, use it. Never import a stub when a real module is available.

Before writing any stub, search the codebase for existing implementations. If the real implementation exists, wire it up. If it doesn't exist, implement it.

---

### 4. APIs must do what they say

Every API endpoint must:
- Accept the correct parameters
- Perform the actual operation
- Return a real result or a meaningful error

**Wrong:**
```python
@router.post("/settings")
async def save_settings():
    return {"status": "ok"}  # Does nothing
```

**Right:** Validate input, persist to database, return accurate confirmation.

---

### 5. Auth and security must be enforced, not skipped

Never create auth infrastructure (JWT, session, middleware) and then fail to apply it. If an endpoint or WebSocket requires authentication, enforce it. Do not leave security checks commented out or bypassed.

**Wrong:**
```python
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await manager.connect(websocket)  # No auth check
```

**Right:** Validate the token before accepting the connection.

---

### 6. Wiring is part of the implementation

Writing a module is not enough. You must also wire it into wherever it's used. An implemented but unconnected module is the same as a stub from the system's point of view.

Checklist before marking something done:
- [ ] The function/class is fully implemented
- [ ] It is imported and called in the correct place
- [ ] All required dependencies (tools, services, config) are passed to it
- [ ] It has been tested at least manually in the running system

---

### 7. Never reference things that don't exist

Do not write config files, role definitions, or system prompts that reference skills, tools, agents, or services that haven't been implemented yet. Every reference must resolve to something real.

**Wrong:**
```yaml
skills:
  - research_and_summarize   # doesn't exist
  - source_evaluation        # doesn't exist
```

**Right:** Only list skills that have corresponding implemented files.

---

### 8. Signature consistency is mandatory

When calling a function or service, always match its actual signature. Never call a function with missing or wrong arguments and assume it will be fixed later — this causes runtime crashes.

Before calling any function, verify its current signature. If the signature needs to change, change it everywhere.

---

## When You're Unsure How to Implement Something

Stop. Ask. Do not stub.

Say: *"I don't know how to implement X yet. Here are the options I see: [list]. Which approach should I take?"*

This is always better than fake code that silently does nothing.

---

## The Standard

Every line of code you write should behave exactly as its name and signature imply — right now, not eventually.
