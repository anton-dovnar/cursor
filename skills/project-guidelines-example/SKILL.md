---
name: project-guidelines-example
description: Project-specific skill template and example guidelines.
---

# Project Guidelines Skill (Example)

This is an example of a project-specific skill. Use this as a template for your own projects.

Based on a real production application: [Zenith](https://zenith.chat) - AI-powered customer discovery platform.

---

## When to Use

Reference this skill when working on the specific project it's designed for. Project skills contain:
- Architecture overview
- File structure
- Code patterns
- Testing requirements
- Deployment workflow

---

## Architecture Overview

**Tech Stack:**
- **Frontend**: Next.js 15 (App Router), TypeScript, React
- **Backend**: FastAPI (Python), Pydantic models
- **Database**: Supabase (PostgreSQL)
- **AI**: Claude API with tool calling and structured output
- **Deployment**: Google Cloud Run
- **Testing**: Playwright (E2E), pytest (backend), React Testing Library

**Services:**
```
┌─────────────────────────────────────────────────────────────┐
│                         Frontend                            │
│  Next.js 15 + TypeScript + TailwindCSS                     │
│  Deployed: Vercel / Cloud Run                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                         Backend                             │
│  FastAPI + Python 3.11 + Pydantic                          │
│  Deployed: Cloud Run                                       │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ Supabase │   │  Claude  │   │  Redis   │
        │ Database │   │   API    │   │  Cache   │
        └──────────┘   └──────────┘   └──────────┘
```

---

## File Structure

```
project/
├── frontend/
│   └── src/
│       ├── app/              # Next.js app router pages
│       │   ├── api/          # API routes
│       │   ├── (auth)/       # Auth-protected routes
│       │   └── workspace/    # Main app workspace
│       ├── components/       # React components
│       │   ├── ui/           # Base UI components
│       │   ├── forms/        # Form components
│       │   └── layouts/      # Layout components
│       ├── hooks/            # Custom React hooks
│       ├── lib/              # Utilities
│       ├── types/            # TypeScript definitions
│       └── config/           # Configuration
│
├── backend/
│   ├── routers/              # FastAPI route handlers
│   ├── models.py             # Pydantic models
│   ├── main.py               # FastAPI app entry
│   ├── auth_system.py        # Authentication
│   ├── database.py           # Database operations
│   ├── services/             # Business logic
│   └── tests/                # pytest tests
│
├── deploy/                   # Deployment configs
├── docs/                     # Documentation
└── scripts/                  # Utility scripts
```

---

## Code Patterns

### API Response Format (FastAPI)

```python
from pydantic import BaseModel
from typing import Generic, TypeVar, Optional

T = TypeVar('T')

class ApiResponse(BaseModel, Generic[T]):
    success: bool
    data: Optional[T] = None
    error: Optional[str] = None

    @classmethod
    def ok(cls, data: T) -> "ApiResponse[T]":
        return cls(success=True, data=data)

    @classmethod
    def fail(cls, error: str) -> "ApiResponse[T]":
        return cls(success=False, error=error)
```

### API Client (Python)

```python
from typing import TypeVar, Optional, Generic
import httpx
from pydantic import BaseModel

T = TypeVar('T')

class ApiResponse(BaseModel, Generic[T]):
    success: bool
    data: Optional[T] = None
    error: Optional[str] = None

class ApiClient:
    def __init__(self, base_url: str):
        self.base_url = base_url
        self.client = httpx.AsyncClient()

    async def fetch(
        self,
        endpoint: str,
        method: str = 'GET',
        **kwargs
    ) -> ApiResponse[T]:
        try:
            url = f'{self.base_url}{endpoint}'
            headers = {
                'Content-Type': 'application/json',
                **kwargs.pop('headers', {})
            }

            response = await self.client.request(
                method=method,
                url=url,
                headers=headers,
                **kwargs
            )

            response.raise_for_status()
            data = response.json()
            return ApiResponse(success=True, data=data)
        except httpx.HTTPStatusError as e:
            return ApiResponse(
                success=False,
                error=f'HTTP {e.response.status_code}'
            )
        except Exception as e:
            return ApiResponse(success=False, error=str(e))

    async def close(self):
        await self.client.aclose()
```

### Claude AI Integration (Structured Output)

```python
from anthropic import Anthropic
from pydantic import BaseModel

class AnalysisResult(BaseModel):
    summary: str
    key_points: list[str]
    confidence: float

async def analyze_with_claude(content: str) -> AnalysisResult:
    client = Anthropic()

    response = client.messages.create(
        model="claude-sonnet-4-5-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": content}],
        tools=[{
            "name": "provide_analysis",
            "description": "Provide structured analysis",
            "input_schema": AnalysisResult.model_json_schema()
        }],
        tool_choice={"type": "tool", "name": "provide_analysis"}
    )

    # Extract tool use result
    tool_use = next(
        block for block in response.content
        if block.type == "tool_use"
    )

    return AnalysisResult(**tool_use.input)
```

### API State Manager (Python)

```python
from typing import TypeVar, Optional, Callable, Awaitable
from dataclasses import dataclass, replace
from enum import Enum

T = TypeVar('T')

class ApiState(Enum):
    IDLE = 'idle'
    LOADING = 'loading'
    SUCCESS = 'success'
    ERROR = 'error'

@dataclass(frozen=True)
class ApiStateManager:
    data: Optional[T] = None
    state: ApiState = ApiState.IDLE
    error: Optional[str] = None

    def set_loading(self) -> 'ApiStateManager':
        return replace(self, state=ApiState.LOADING, error=None)

    def set_success(self, data: T) -> 'ApiStateManager':
        return replace(
            self,
            data=data,
            state=ApiState.SUCCESS,
            error=None
        )

    def set_error(self, error: str) -> 'ApiStateManager':
        return replace(
            self,
            data=None,
            state=ApiState.ERROR,
            error=error
        )

class ApiExecutor:
    def __init__(self, fetch_fn: Callable[[], Awaitable[ApiResponse[T]]]):
        self.fetch_fn = fetch_fn
        self.state = ApiStateManager()

    async def execute(self) -> ApiStateManager:
        self.state = self.state.set_loading()

        result = await self.fetch_fn()

        if result.success:
            self.state = self.state.set_success(result.data)
        else:
            self.state = self.state.set_error(result.error or 'Unknown error')

        return self.state
```

---

## Testing Requirements

### Backend (pytest)

```bash
# Run all tests
poetry run pytest tests/

# Run with coverage
poetry run pytest tests/ --cov=. --cov-report=html

# Run specific test file
poetry run pytest tests/test_auth.py -v
```

**Test structure:**
```python
import pytest
from httpx import AsyncClient
from main import app

@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

@pytest.mark.asyncio
async def test_health_check(client: AsyncClient):
    response = await client.get("/health")
    assert response.status_code == 200
    assert response.json()["status"] == "healthy"
```

### Service Layer Tests (Python)

```bash
# Run tests
pytest tests/

# Run with coverage
pytest tests/ --cov=. --cov-report=html

# Run E2E tests
pytest tests/e2e/
```

**Test structure:**
```python
import pytest
from unittest.mock import AsyncMock, patch
from app.services.workspace_service import WorkspaceService

@pytest.fixture
def workspace_service():
    return WorkspaceService()

def test_workspace_service_initialization(workspace_service):
    assert workspace_service is not None

@pytest.mark.asyncio
async def test_creates_session_successfully(workspace_service):
    with patch('app.services.workspace_service.create_session') as mock_create:
        mock_create.return_value = {'id': 'session-123', 'status': 'created'}

        result = await workspace_service.create_session('user-123')

        assert result['status'] == 'created'
        mock_create.assert_called_once_with('user-123')
```

---

## Deployment Workflow

### Pre-Deployment Checklist

- [ ] All tests passing locally
- [ ] `npm run build` succeeds (frontend)
- [ ] `poetry run pytest` passes (backend)
- [ ] No hardcoded secrets
- [ ] Environment variables documented
- [ ] Database migrations ready

### Deployment Commands

```bash
# Build and deploy frontend
cd frontend && npm run build
gcloud run deploy frontend --source .

# Build and deploy backend
cd backend
gcloud run deploy backend --source .
```

### Environment Variables

```bash
# Frontend (.env.local)
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...

# Backend (.env)
DATABASE_URL=postgresql://...
ANTHROPIC_API_KEY=sk-ant-...
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_KEY=eyJ...
```

---

## Critical Rules

1. **No emojis** in code, comments, or documentation
2. **Immutability** - never mutate objects or arrays
3. **TDD** - write tests before implementation
4. **80% coverage** minimum
5. **Many small files** - 200-400 lines typical, 800 max
6. **No console.log** in production code
7. **Proper error handling** with try/catch
8. **Input validation** with Pydantic/Zod

---

## Related Skills

- `coding-standards.md` - General coding best practices
- `backend-patterns.md` - API and database patterns
- `frontend-patterns.md` - React and Next.js patterns
- `tdd-workflow/` - Test-driven development methodology
