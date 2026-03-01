# EARS (Easy Approach to Requirements Syntax) Format

Requirements MUST use one of these 5 patterns. No free-form text.

## Patterns

### 1. Ubiquitous (always active)
```
The [system] shall [action].
```
Example: The system shall encrypt all stored passwords using bcrypt.

### 2. Event-Driven (triggered by event)
```
When [event], the [system] shall [action].
```
Example: When a user submits the login form, the system shall validate credentials within 2 seconds.

### 3. State-Driven (while condition holds)
```
While [state], the [system] shall [action].
```
Example: While the server is in maintenance mode, the system shall return HTTP 503 to all API requests.

### 4. Optional (feature-gated)
```
Where [feature is enabled], the [system] shall [action].
```
Example: Where dark mode is enabled, the system shall apply the dark color palette to all UI components.

### 5. Unwanted (negative / safety)
```
If [condition], then the [system] shall [action] (to prevent [consequence]).
```
Example: If login attempts exceed 5 in 10 minutes, then the system shall lock the account for 30 minutes (to prevent brute-force attacks).

## Rules

- Each requirement gets exactly ONE pattern
- Quantify where possible (time, count, size)
- No ambiguous words: "appropriate", "reasonable", "user-friendly"
- One action per requirement (split compound actions)
