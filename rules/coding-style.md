---
description: "Coding standards: immutability, file organization, error handling, input validation, and code quality checklist"
alwaysApply: false
---

# Coding Style

## Immutability (CRITICAL)

ALWAYS create new objects, NEVER mutate:

```python
# ❌ WRONG: Mutating input objects
def update_user(user: dict, name: str):
    user["name"] = name   # MUTATION!
    return user

# ✅ CORRECT: Return a new object
def update_user(user: dict, name: str):
    return {
        **user,
        "name": name,
    }
```

## File Organization

MANY SMALL FILES > FEW LARGE FILES:
- High cohesion, low coupling
- 200-400 lines typical, 800 max
- Avoid “god modules”
- Extract utilities from large components
- Organize by feature/domain, not by type

## Error Handling

ALWAYS handle errors comprehensively:

```python
# ❌ WRONG: Silent failure
result = risky_operation()  # may throw

# ✅ CORRECT: Explicit error handling
import logging

logger = logging.getLogger(__name__)

def safe_operation():
    try:
        return risky_operation()
    except Exception as exc:
        logger.error("Operation failed: %s", exc)
        raise RuntimeError("Something went wrong while processing your request") from exc
```

## Input Validation

ALWAYS validate user input:

```python
from pydantic import BaseModel, EmailStr, conint

class UserInput(BaseModel):
    email: EmailStr
    age: conint(ge=0, le=150)

validated = UserInput(**input_data)

# Example with Django REST Framework:
from rest_framework import serializers

class UserSerializer(serializers.Serializer):
    email = serializers.EmailField()
    age = serializers.IntegerField(min_value=0, max_value=150)

validated = UserSerializer(data=input_data)
validated.is_valid(raise_exception=True)
```

## Code Quality Checklist

Before marking work complete:
- [ ] Code is readable and well-named
- [ ] Functions are small (<50 lines)
- [ ] Files are focused (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Proper error handling
- [ ] No print() debugging (use logging)
- [ ] No hardcoded values (use settings/env)
- [ ] No mutation (immutable patterns used)
- [ ] Type hints added where meaningful
- [ ] Docstrings for public functions/classes
