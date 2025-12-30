# Development Workflow Guide

**Version:** 1.0
**Last Updated:** 2025-12-29
**Target Audience:** All Developers (Frontend, Backend, Full-Stack)

This guide provides comprehensive workflows and best practices for developing the Weekly Planner project. It covers git submodules, API development, testing, and documentation standards.

---

## Table of Contents

1. [Project Structure](#project-structure)
2. [Git Submodule Management](#git-submodule-management)
3. [Development Environment Setup](#development-environment-setup)
4. [API Development Workflow](#api-development-workflow)
5. [Commit Message Convention](#commit-message-convention)
6. [Testing Strategy](#testing-strategy)
7. [Documentation Standards](#documentation-standards)
8. [Common Tasks and Commands](#common-tasks-and-commands)
9. [Debugging Tips](#debugging-tips)
10. [Performance Optimization](#performance-optimization)
11. [Troubleshooting](#troubleshooting)

---

## Project Structure

The Weekly Planner project consists of three repositories:

```
weekly-planner/
├── weekly-planner-frontend/     # React + TypeScript frontend
│   ├── docs/                    # Git submodule → weekly-planner-docs
│   ├── src/
│   ├── package.json
│   └── ...
│
├── weekly-planner-backend/      # Spring Boot backend
│   ├── docs/                    # Git submodule → weekly-planner-docs
│   ├── src/
│   ├── pom.xml
│   └── ...
│
└── weekly-planner-docs/         # Shared documentation (standalone)
    ├── domain-model.md
    ├── api-contract.md
    ├── business-rules.md
    ├── ui-spec.md
    ├── backend-integration-guide.md
    ├── development-workflow.md (this file)
    └── ...
```

**Repository Relationships:**
- `weekly-planner-docs` is a standalone repository
- Both frontend and backend repositories include `docs/` as a git submodule
- Shared documentation ensures consistency across all implementations

---

## Git Submodule Management

### Understanding Submodules

Git submodules allow you to include one repository inside another as a subdirectory. In this project, the `docs/` directory is a submodule that points to the `weekly-planner-docs` repository.

### Initial Setup

**When cloning a repository with submodules:**

```bash
# Option 1: Clone with submodules in one command
git clone --recurse-submodules https://github.com/{username}/weekly-planner-frontend.git

# Option 2: Clone first, then initialize submodules
git clone https://github.com/{username}/weekly-planner-frontend.git
cd weekly-planner-frontend
git submodule init
git submodule update
```

**Adding the submodule (already done, for reference):**

```bash
# From frontend or backend repository root
git submodule add https://github.com/{username}/weekly-planner-docs.git docs
git commit -m "chore: Add docs submodule"
```

### Daily Workflow with Submodules

#### 1. Reading Documentation

Documentation is always in sync with the submodule commit reference:

```bash
# View current submodule status
git submodule status

# Example output:
# a1b2c3d4 docs (heads/main)
```

Just read files in `docs/` as normal:

```bash
cat docs/api-contract.md
code docs/domain-model.md
```

#### 2. Updating Documentation

When you need to modify shared documentation:

```bash
# Step 1: Navigate to submodule directory
cd docs/

# Step 2: Ensure you're on the main branch
git checkout main
git pull origin main

# Step 3: Make your changes
vim development-workflow.md

# Step 4: Commit in the submodule
git add development-workflow.md
git commit -m "docs: Add deployment section to workflow guide"
git push origin main

# Step 5: Return to parent repository
cd ..

# Step 6: Commit the submodule reference update
git add docs
git commit -m "chore: Update docs submodule reference"
git push origin main
```

**Important:**
- Always commit changes inside the `docs/` directory first
- Then commit the submodule reference update in the parent repo
- This ensures the parent repo points to the correct commit

#### 3. Pulling Changes from Others

When someone else updates the documentation:

```bash
# In parent repository
git pull origin main

# Update submodules to match the new reference
git submodule update --remote docs

# Or update all submodules
git submodule update --remote
```

**Automatic update on pull:**

```bash
# Set git to automatically update submodules when pulling
git config submodule.recurse true

# Now this will update submodules automatically
git pull
```

### Common Submodule Commands

```bash
# Check submodule status
git submodule status

# Show all submodules
git submodule

# Update submodule to latest remote commit
git submodule update --remote docs

# Update all submodules
git submodule update --remote --merge

# Reset submodule to parent's reference
git submodule update --init

# Remove submodule (rarely needed)
git submodule deinit docs
git rm docs
rm -rf .git/modules/docs
```

### Submodule Workflow Example

**Scenario:** Adding a new API endpoint

```bash
# 1. Update API contract in docs
cd docs/
git checkout main
git pull

# Edit api-contract.md
vim api-contract.md

# Commit changes
git add api-contract.md
git commit -m "docs: Add GET /tasks/search endpoint"
git push origin main

# 2. Return to parent repo and update reference
cd ..
git add docs
git commit -m "chore: Update docs submodule reference"

# 3. Implement the endpoint in backend
# (make changes to backend code)
git add src/
git commit -m "feat: Implement task search endpoint"

# 4. Push all changes
git push origin main
```

### Troubleshooting Submodules

**Problem:** Submodule shows modified but you didn't change anything

```bash
# Check what changed
cd docs/
git status
git diff

# If it's just a commit reference, reset it
git checkout .
cd ..
git submodule update --init
```

**Problem:** Submodule is in detached HEAD state

```bash
cd docs/
git checkout main
git pull origin main
cd ..
git add docs
git commit -m "chore: Update docs submodule to main branch"
```

**Problem:** Submodule directory is empty after clone

```bash
git submodule init
git submodule update
```

**Problem:** Submodule conflicts during merge

```bash
# Accept the incoming submodule reference
git checkout --theirs docs
git add docs

# Or accept your current reference
git checkout --ours docs
git add docs

# Then complete the merge
git commit
```

---

## Development Environment Setup

### Frontend Setup

**Prerequisites:**
- Node.js 18+ and npm 8+
- Git
- Code editor (VS Code recommended)

**Initial Setup:**

```bash
# 1. Clone with submodules
git clone --recurse-submodules https://github.com/{username}/weekly-planner-frontend.git
cd weekly-planner-frontend

# 2. Install dependencies
npm install

# 3. Create environment file
cp .env.example .env

# 4. Edit .env with your settings
code .env
```

**.env Configuration:**

```env
# Backend API URL
VITE_API_URL=http://localhost:8080/api/v1

# Optional: Enable debug mode
VITE_DEBUG=true
```

**Running Frontend:**

```bash
# Development server (http://localhost:3000)
npm run dev

# Production build
npm run build

# Preview production build
npm run preview

# Lint check
npm run lint

# Format code
npm run format
```

### Backend Setup

**Prerequisites:**
- Java 17+
- Maven or Gradle
- Git
- IDE (IntelliJ IDEA recommended)

**Initial Setup:**

```bash
# 1. Clone with submodules
git clone --recurse-submodules https://github.com/{username}/weekly-planner-backend.git
cd weekly-planner-backend

# 2. Create application.yml
cp src/main/resources/application.yml.example src/main/resources/application.yml

# 3. Edit configuration
code src/main/resources/application.yml

# 4. Install dependencies
./mvnw clean install
```

**application.yml Configuration:**

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:

  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

jwt:
  secret: your-secret-key-here
  expiration: 86400000  # 24 hours
```

**Running Backend:**

```bash
# Development mode
./mvnw spring-boot:run

# Build JAR
./mvnw clean package

# Run JAR
java -jar target/weekly-planner-backend-0.0.1-SNAPSHOT.jar

# Run tests
./mvnw test
```

### Running Both Services

**Option 1: Separate Terminals**

```bash
# Terminal 1: Backend
cd weekly-planner-backend
./mvnw spring-boot:run

# Terminal 2: Frontend
cd weekly-planner-frontend
npm run dev
```

**Option 2: Using Docker Compose (if available)**

```bash
docker-compose up
```

**Option 3: VS Code Tasks (frontend only)**

Create `.vscode/tasks.json`:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Run Frontend",
      "type": "shell",
      "command": "npm run dev",
      "problemMatcher": []
    }
  ]
}
```

### Verifying Setup

1. Backend running: http://localhost:8080/actuator/health (if actuator enabled)
2. Frontend running: http://localhost:3000
3. API accessible: http://localhost:8080/api/v1/auth/login

**Quick Test:**

```bash
# Test backend health
curl http://localhost:8080/api/v1/auth/login \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Should return 401 or user not found if backend is working
```

---

## API Development Workflow

### Adding a New Endpoint

Follow these steps when adding a new API endpoint:

#### Step 1: Design and Document

**Update `docs/api-contract.md`:**

```bash
cd docs/
git checkout main
git pull
```

Add your endpoint specification:

```markdown
### GET /api/v1/tasks/search

Search tasks across all plans.

**Query Parameters:**
- `query` (string, required) - Search query
- `status` (string, optional) - Filter by status
- `page` (integer, optional) - Page number (default: 0)
- `size` (integer, optional) - Page size (default: 20)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "content": [...],
    "page": 0,
    "size": 20,
    "totalElements": 50,
    "totalPages": 3
  }
}
```

Commit the documentation:

```bash
git add api-contract.md
git commit -m "docs: Add task search endpoint specification"
git push origin main
cd ..
git add docs
git commit -m "chore: Update docs submodule reference"
```

#### Step 2: Implement Backend

**Create/Update Controller:**

```java
@RestController
@RequestMapping("/api/v1/tasks")
public class TaskController {

    @GetMapping("/search")
    public ResponseEntity<ApiResponse<Page<TaskDto>>> searchTasks(
        @RequestParam String query,
        @RequestParam(required = false) TaskStatus status,
        @Pageable pageable
    ) {
        Page<TaskDto> results = taskService.search(query, status, pageable);
        return ResponseEntity.ok(ApiResponse.success(results));
    }
}
```

**Write Service Layer:**

```java
@Service
public class TaskService {

    public Page<TaskDto> search(String query, TaskStatus status, Pageable pageable) {
        // Implementation
    }
}
```

**Commit Backend Changes:**

```bash
git add src/
git commit -m "feat: Implement task search endpoint"
git push origin main
```

#### Step 3: Test Backend

```bash
# Run backend tests
./mvnw test

# Manual test with curl
curl "http://localhost:8080/api/v1/tasks/search?query=meeting" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

#### Step 4: Implement Frontend

**Update API Client:**

```typescript
// src/api/tasks.ts
export const taskApi = {
  // ... existing methods

  search: (query: string, status?: TaskStatus, page = 0, size = 20) =>
    apiClient.get<ApiResponse<PaginatedResponse<Task>>>('/tasks/search', {
      params: { query, status, page, size }
    }),
};
```

**Update Types (if needed):**

```typescript
// src/types/task.ts
export interface TaskSearchRequest {
  query: string;
  status?: TaskStatus;
  page?: number;
  size?: number;
}
```

**Use in Component:**

```typescript
// src/pages/TaskSearch.tsx
const { data, isLoading } = useQuery(
  ['tasks', 'search', query],
  () => taskApi.search(query, status)
);
```

**Commit Frontend Changes:**

```bash
git add src/
git commit -m "feat: Add task search functionality"
git push origin main
```

#### Step 5: Integration Testing

1. Start both backend and frontend
2. Test the feature end-to-end
3. Check browser console for errors
4. Verify network requests in DevTools
5. Test edge cases (empty results, errors, etc.)

### Modifying Existing Endpoints

**Breaking Changes (requires coordination):**

1. **Document the change** in `docs/api-contract.md`
   - Mark old version as deprecated
   - Add migration notes

2. **Version the endpoint** (if major breaking change)
   ```
   /api/v1/tasks  → /api/v2/tasks
   ```

3. **Update backend** with backward compatibility (if possible)

4. **Update frontend** to use new format

5. **Deprecation period** (optional)
   - Keep old endpoint working
   - Log warnings when used
   - Remove after migration period

**Non-Breaking Changes (additive):**

1. Add new optional fields/parameters
2. Update documentation
3. Implement backend changes
4. Update frontend to use new features (optional)

### API Versioning Strategy

**Current Approach:** `/api/v1/...`

**When to increment version:**
- Breaking changes to request/response format
- Removal of required fields
- Changed authentication method
- Major behavioral changes

**How to handle versions:**
```java
@RestController
@RequestMapping("/api/v1")
public class TaskControllerV1 { }

@RestController
@RequestMapping("/api/v2")
public class TaskControllerV2 { }
```

---

## Commit Message Convention

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type

Must be one of the following:

- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation only changes
- **style**: Code style changes (formatting, semicolons, etc.)
- **refactor**: Code refactoring (no feature change or bug fix)
- **perf**: Performance improvement
- **test**: Adding or updating tests
- **build**: Changes to build system or dependencies
- **ci**: Changes to CI configuration
- **chore**: Other changes (updating submodules, etc.)
- **revert**: Revert a previous commit

### Scope (Optional)

The scope specifies what part of the codebase is affected:

**Frontend:**
- `auth` - Authentication
- `task` - Task management
- `plan` - Weekly planning
- `review` - Weekly review
- `notification` - Notifications
- `ui` - UI components
- `api` - API client
- `store` - State management

**Backend:**
- `auth` - Authentication
- `task` - Task service
- `plan` - Plan service
- `review` - Review service
- `notification` - Notification service
- `security` - Security configuration
- `database` - Database/JPA changes

**Shared:**
- `docs` - Documentation submodule
- `config` - Configuration files
- `deps` - Dependencies

### Subject

- Use imperative, present tense ("add" not "added" or "adds")
- Don't capitalize first letter
- No period at the end
- Maximum 50 characters

### Body (Optional)

- Use imperative, present tense
- Explain **what** and **why**, not **how**
- Wrap at 72 characters

### Footer (Optional)

- Reference issues: `Fixes #123`
- Breaking changes: `BREAKING CHANGE: describe what broke`

### Examples

**Good Commit Messages:**

```
feat(task): add task search functionality

Implement full-text search across tasks with filtering
by status and priority. Supports pagination.

Closes #45
```

```
fix(auth): correct JWT token validation

Token expiration was not being checked properly,
allowing expired tokens to authenticate.

Fixes #78
```

```
docs: add API development workflow guide

Add comprehensive guide for adding new endpoints
including documentation, implementation, and testing.
```

```
refactor(plan): extract plan calculation logic

Move complex date calculations into separate utility
class for better testability and reusability.
```

```
chore: update docs submodule reference

Pull latest API contract changes from docs repository.
```

```
test(task): add integration tests for task creation

Add tests covering task creation with various
reminder and priority configurations.
```

**Bad Commit Messages:**

```
❌ Fixed bug
❌ WIP
❌ Updated files
❌ asdfasdf
❌ fix: Fixed the authentication bug that was broken before
```

### Commit Message Templates

Set up a commit message template:

```bash
# Create template file
cat > ~/.gitmessage << EOF
# <type>(<scope>): <subject>
#
# <body>
#
# <footer>
#
# Type: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert
# Scope: auth, task, plan, review, notification, ui, api, store, security, database, docs, config, deps
# Subject: imperative, lowercase, no period, max 50 chars
# Body: wrap at 72 chars, explain what and why
# Footer: reference issues (Fixes #123), breaking changes
EOF

# Configure git to use it
git config --global commit.template ~/.gitmessage
```

---

## Testing Strategy

### Frontend Testing

#### Unit Tests (Component Level)

**Tools:** Vitest + React Testing Library

```typescript
// src/components/task/TaskItem.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { TaskItem } from './TaskItem';

describe('TaskItem', () => {
  const mockTask = {
    id: 'task_1',
    title: 'Test Task',
    status: 'PENDING',
    priority: 'HIGH',
  };

  it('renders task title', () => {
    render(<TaskItem task={mockTask} />);
    expect(screen.getByText('Test Task')).toBeInTheDocument();
  });

  it('toggles completion status', () => {
    const onStatusChange = vi.fn();
    render(<TaskItem task={mockTask} onStatusChange={onStatusChange} />);

    const checkbox = screen.getByRole('checkbox');
    fireEvent.click(checkbox);

    expect(onStatusChange).toHaveBeenCalledWith('COMPLETED');
  });
});
```

**Run Tests:**

```bash
npm run test
npm run test:watch
npm run test:coverage
```

#### Integration Tests (API Level)

```typescript
// src/api/tasks.test.ts
import { taskApi } from './tasks';
import { server } from '../mocks/server';
import { rest } from 'msw';

describe('Task API', () => {
  it('creates task successfully', async () => {
    const result = await taskApi.create('plan_1', '2024-01-01', {
      title: 'New Task',
      priority: 'HIGH',
    });

    expect(result.success).toBe(true);
    expect(result.data.title).toBe('New Task');
  });

  it('handles creation error', async () => {
    server.use(
      rest.post('/api/v1/plans/:planId/tasks', (req, res, ctx) => {
        return res(ctx.status(400), ctx.json({ error: 'Invalid data' }));
      })
    );

    await expect(taskApi.create('plan_1', '2024-01-01', {}))
      .rejects.toThrow();
  });
});
```

#### E2E Tests (User Flow)

**Tools:** Playwright or Cypress

```typescript
// e2e/task-management.spec.ts
import { test, expect } from '@playwright/test';

test('user can create and complete task', async ({ page }) => {
  // Login
  await page.goto('http://localhost:3000/login');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('button[type="submit"]');

  // Navigate to Today page
  await page.click('text=Today');
  await expect(page).toHaveURL(/.*today/);

  // Add task
  await page.click('text=Add Task');
  await page.fill('[name="title"]', 'Test Task');
  await page.click('text=Save');

  // Verify task appears
  await expect(page.locator('text=Test Task')).toBeVisible();

  // Complete task
  await page.check('[role="checkbox"]');
  await expect(page.locator('text=Test Task')).toHaveClass(/line-through/);
});
```

### Backend Testing

#### Unit Tests (Service Layer)

```java
@SpringBootTest
class TaskServiceTest {

    @Autowired
    private TaskService taskService;

    @MockBean
    private TaskRepository taskRepository;

    @Test
    void createTask_Success() {
        // Given
        CreateTaskRequest request = CreateTaskRequest.builder()
            .title("Test Task")
            .priority(Priority.HIGH)
            .build();

        // When
        TaskDto result = taskService.createTask("plan_1", "2024-01-01", request);

        // Then
        assertNotNull(result);
        assertEquals("Test Task", result.getTitle());
        assertEquals(Priority.HIGH, result.getPriority());
    }

    @Test
    void createTask_InvalidDate_ThrowsException() {
        // Given
        CreateTaskRequest request = CreateTaskRequest.builder()
            .title("Test Task")
            .build();

        // When/Then
        assertThrows(InvalidDateException.class, () ->
            taskService.createTask("plan_1", "invalid-date", request)
        );
    }
}
```

#### Integration Tests (API Layer)

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class TaskControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @WithMockUser
    void createTask_Success() throws Exception {
        // Given
        CreateTaskRequest request = CreateTaskRequest.builder()
            .title("Test Task")
            .priority(Priority.HIGH)
            .build();

        // When/Then
        mockMvc.perform(post("/api/v1/plans/{planId}/tasks", "plan_1")
                .param("date", "2024-01-01")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.success").value(true))
            .andExpect(jsonPath("$.data.title").value("Test Task"));
    }
}
```

#### Repository Tests

```java
@DataJpaTest
class TaskRepositoryTest {

    @Autowired
    private TaskRepository taskRepository;

    @Test
    void findByPlanIdAndDate_ReturnsTasksForDate() {
        // Given
        LocalDate date = LocalDate.of(2024, 1, 1);
        Task task = Task.builder()
            .planId("plan_1")
            .date(date)
            .title("Test Task")
            .build();
        taskRepository.save(task);

        // When
        List<Task> results = taskRepository.findByPlanIdAndDate("plan_1", date);

        // Then
        assertEquals(1, results.size());
        assertEquals("Test Task", results.get(0).getTitle());
    }
}
```

### API Testing Tools

#### Using test-api.js

```bash
# Navigate to frontend directory
cd weekly-planner-frontend

# Run API test script
node test-api.js
```

**What it tests:**
- Authentication flow
- Token validation
- Critical endpoints
- Fallback logic for missing endpoints

**Example output:**
```
=== API Endpoint Test ===

1. Testing Login...
✅ Login successful
   Token: eyJhbGciOiJIUzI1NiIs...

2. Testing /auth/me...
✅ /auth/me successful

3. Testing /plans/current...
⚠️ /plans/current not found, using fallback
✅ Fallback successful
```

#### Using test-api-endpoints.html

```bash
# Open in browser
open test-api-endpoints.html  # macOS
start test-api-endpoints.html  # Windows
xdg-open test-api-endpoints.html  # Linux
```

**Features:**
- Visual test results
- Real-time request/response display
- Highlights missing endpoints
- CORS error detection

#### Manual Testing Checklist

**Authentication:**
- [ ] User registration with valid data
- [ ] Registration with duplicate email (should fail)
- [ ] Login with correct credentials
- [ ] Login with incorrect password (should fail)
- [ ] Access protected endpoint with token
- [ ] Access protected endpoint without token (should return 401)
- [ ] Token expiration handling

**Task Management:**
- [ ] Create task with required fields only
- [ ] Create task with all optional fields
- [ ] Create task with invalid data (should fail)
- [ ] Update task title
- [ ] Update task status
- [ ] Move task to another day
- [ ] Delete task
- [ ] List tasks for a specific date

**Weekly Planning:**
- [ ] Create new weekly plan
- [ ] Get current week's plan
- [ ] Confirm plan (status changes to CONFIRMED)
- [ ] Add task after confirmation (creates changelog)
- [ ] Update daily memo

**Review:**
- [ ] Get weekly review with statistics
- [ ] View completion rate chart
- [ ] View change history timeline
- [ ] Filter changes by type

**Notifications:**
- [ ] List notifications
- [ ] Get unread count
- [ ] Mark single notification as read
- [ ] Mark all notifications as read

### Test Coverage Goals

**Frontend:**
- Unit tests: 70%+ coverage
- Integration tests: Critical user flows
- E2E tests: Main happy paths

**Backend:**
- Unit tests: 80%+ coverage
- Integration tests: All API endpoints
- Repository tests: All custom queries

---

## Documentation Standards

### Where Documentation Goes

**Main Repository (Frontend/Backend):**
- `README.md` - Project overview, quick start
- `CLAUDE.md` - AI assistant instructions, project-specific context
- Code comments - Implementation details
- JSDoc/Javadoc - API documentation in code

**Docs Submodule (Shared):**
- `domain-model.md` - Domain entities, enums, relationships
- `api-contract.md` - REST API specification
- `business-rules.md` - Business logic rules
- `ui-spec.md` - UI/UX specifications
- `backend-integration-guide.md` - Backend integration checklist
- `development-workflow.md` - This file

### Documentation Update Workflow

**When to update docs submodule:**

1. **API Changes** → Update `api-contract.md`
2. **Domain Model Changes** → Update `domain-model.md`
3. **Business Rules** → Update `business-rules.md`
4. **UI Changes** → Update `ui-spec.md`
5. **Integration Changes** → Update `backend-integration-guide.md`

**When to update main repository docs:**

1. **Setup instructions change** → Update `README.md`

### Code Example Guidelines

**IMPORTANT: Reference actual implementation instead of duplicating code**

When documenting code in markdown files:

1. **Don't duplicate long code examples**
   ```markdown
   ❌ BAD: Copying entire function implementation
   ✅ GOOD: Reference with line numbers
   ```

2. **Use file references with line numbers**
   ```markdown
   // Full implementation at:
   // src/stores/authStore.ts:14-26
   export const useAuthStore = create<AuthState>()(
     persist(/* ... */)
   )
   ```

3. **Show only essential signatures**
   ```markdown
   // See implementation: src/api/tasks.ts:4-38
   export const taskApi = {
     create: (planId: string, date: string, data: CreateTaskRequest) => Promise<Task>
     update: (planId: string, taskId: string, data: UpdateTaskRequest) => Promise<Task>
     // ... other methods
   }
   ```

4. **Benefits of this approach**
   - Single source of truth (actual code)
   - Documentation stays up-to-date
   - Smaller file sizes
   - Easier to maintain
   - IDE can navigate to actual files
2. **Project-specific context** → Update `CLAUDE.md`
3. **Build process changes** → Update `README.md`

### API Documentation Format

Follow this template in `api-contract.md`:

```markdown
### GET /api/v1/resource/{id}

Brief description of what this endpoint does.

**Path Parameters:**
- `id` (string, required) - Resource identifier

**Query Parameters:**
- `include` (string, optional) - Related resources to include

**Request Headers:**
- `Authorization` (string, required) - Bearer token

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "resource_123",
    "name": "Resource Name"
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid input
- `401 Unauthorized` - Missing or invalid token
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server error

**Example:**
```bash
curl http://localhost:8080/api/v1/resource/123 \
  -H "Authorization: Bearer TOKEN"
```
```

### Code Comments

**When to comment:**
- Complex algorithms
- Non-obvious business logic
- Workarounds for bugs/limitations
- Public APIs

**When NOT to comment:**
- Self-explanatory code
- Obvious operations
- Redundant descriptions

**Good Comments:**

```typescript
// Calculate departure time by subtracting total commute duration
// from arrival time, accounting for timezone differences
const departureTime = calculateDepartureTime(arrivalTime, totalMinutes);
```

```java
// JWT tokens are stored in localStorage for persistence across sessions.
// This is acceptable for this app but consider httpOnly cookies for
// higher security requirements.
```

**Bad Comments:**

```typescript
// Set loading to true
setLoading(true);

// Loop through tasks
tasks.forEach(task => { ... });
```

### JSDoc / Javadoc

**Frontend (TypeScript/JSDoc):**

```typescript
/**
 * Creates a new task in the specified plan.
 *
 * @param planId - The ID of the weekly plan
 * @param date - The date for the task (YYYY-MM-DD)
 * @param data - Task creation data
 * @returns Promise resolving to the created task
 * @throws {ApiError} If the request fails
 *
 * @example
 * const task = await taskApi.create('plan_123', '2024-01-01', {
 *   title: 'Project Meeting',
 *   priority: 'HIGH'
 * });
 */
export const create = (
  planId: string,
  date: string,
  data: CreateTaskRequest
): Promise<ApiResponse<Task>> => { ... }
```

**Backend (Javadoc):**

```java
/**
 * Creates a new task in the specified weekly plan.
 *
 * @param planId the ID of the weekly plan
 * @param date the date for the task
 * @param request the task creation request
 * @return the created task DTO
 * @throws PlanNotFoundException if the plan doesn't exist
 * @throws InvalidDateException if the date is outside the plan's week
 */
public TaskDto createTask(String planId, LocalDate date, CreateTaskRequest request) {
    // Implementation
}
```

### Required Documentation for New Features

When adding a new feature, you must provide:

1. **API Contract** (in `docs/api-contract.md`)
   - All new endpoints
   - Request/response formats
   - Error cases

2. **Domain Model** (in `docs/domain-model.md`, if applicable)
   - New entities
   - New enums
   - Relationship changes

3. **UI Specification** (in `docs/ui-spec.md`, if applicable)
   - New screens/components
   - User interactions
   - Visual design notes

4. **Code Comments**
   - Complex logic explanation
   - Public API documentation

5. **Tests**
   - Unit tests for new code
   - Integration tests for new endpoints
   - E2E tests for new user flows

---

## Common Tasks and Commands

### Git Operations

```bash
# Daily workflow
git pull                          # Update from remote
git submodule update --remote     # Update submodules
git status                        # Check status
git add .                         # Stage all changes
git commit -m "feat: add feature" # Commit
git push                          # Push to remote

# Branch operations
git checkout -b feature/task-search  # Create feature branch
git checkout main                    # Switch to main
git merge feature/task-search        # Merge feature branch
git branch -d feature/task-search    # Delete merged branch

# Submodule operations
cd docs/                          # Enter submodule
git checkout main                 # Switch to main branch
git pull origin main              # Update docs
# Make changes
git add .
git commit -m "docs: update API"
git push origin main
cd ..                             # Return to parent repo
git add docs                      # Stage submodule reference
git commit -m "chore: update docs submodule"
git push

# Stashing changes
git stash                         # Stash uncommitted changes
git stash pop                     # Restore stashed changes
git stash list                    # List all stashes

# View history
git log --oneline                 # Compact log
git log --graph --all             # Visual branch history
git log docs/                     # Submodule commit history

# Undo operations
git reset --soft HEAD~1           # Undo last commit, keep changes
git reset --hard HEAD~1           # Undo last commit, discard changes
git checkout -- file.txt          # Discard file changes
```

### Frontend Commands

```bash
# Development
npm run dev                       # Start dev server
npm run build                     # Production build
npm run preview                   # Preview production build

# Code quality
npm run lint                      # Run ESLint
npm run lint:fix                  # Fix linting errors
npm run format                    # Format with Prettier
npm run type-check                # TypeScript type checking

# Testing
npm run test                      # Run tests once
npm run test:watch                # Run tests in watch mode
npm run test:coverage             # Generate coverage report

# Dependencies
npm install                       # Install all dependencies
npm install package-name          # Install new package
npm update                        # Update dependencies
npm outdated                      # Check for outdated packages

# Cleanup
rm -rf node_modules package-lock.json
npm install                       # Fresh install

# Build analysis
npm run build -- --report         # Analyze bundle size
```

### Backend Commands

```bash
# Development
./mvnw spring-boot:run            # Start Spring Boot app
./mvnw clean install              # Build and install
./mvnw package                    # Build JAR

# Testing
./mvnw test                       # Run all tests
./mvnw test -Dtest=TaskServiceTest  # Run specific test
./mvnw verify                     # Run tests + integration tests

# Dependencies
./mvnw dependency:tree            # Show dependency tree
./mvnw versions:display-dependency-updates  # Check for updates

# Code quality
./mvnw checkstyle:check           # Run Checkstyle
./mvnw spotless:check             # Check code formatting
./mvnw spotless:apply             # Apply code formatting

# Database
./mvnw flyway:migrate             # Run database migrations
./mvnw flyway:info                # Show migration status

# Cleanup
./mvnw clean                      # Clean build directory
```

### Testing Commands

```bash
# Frontend
npm run test                      # Run all tests
npm run test:watch                # Watch mode
npm run test:ui                   # Open test UI
npm run test:coverage             # Generate coverage

# Backend
./mvnw test                       # Unit tests
./mvnw verify                     # Integration tests
./mvnw test jacoco:report         # Generate coverage report

# API testing
node test-api.js                  # Run API test script
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# E2E testing
npx playwright test               # Run E2E tests
npx playwright test --ui          # Run with UI
npx playwright codegen            # Generate tests
```

### Database Commands

```bash
# H2 Console (if enabled)
# Navigate to: http://localhost:8080/h2-console
# JDBC URL: jdbc:h2:mem:testdb

# PostgreSQL (if using)
psql -U postgres                  # Connect to PostgreSQL
\l                                # List databases
\c weekly_planner                 # Connect to database
\dt                               # List tables
\d task                           # Describe table

# Migration
./mvnw flyway:migrate             # Run migrations
./mvnw flyway:info                # Show migration status
./mvnw flyway:clean               # Clean database (dev only!)
```

### Docker Commands

```bash
# If using Docker for database
docker-compose up -d              # Start services in background
docker-compose down               # Stop services
docker-compose logs -f            # View logs
docker-compose ps                 # List running services

# Database container
docker exec -it postgres psql -U postgres  # Connect to DB
```

---

## Debugging Tips

### Frontend Debugging

#### Browser DevTools

**Console:**
- Check for JavaScript errors
- View API response data
- Test code snippets

```javascript
// In browser console
localStorage.getItem('auth-storage')  // Check stored auth
JSON.parse(localStorage.getItem('auth-storage'))  // Parse auth data
```

**Network Tab:**
- Monitor API requests/responses
- Check request headers (Authorization)
- Verify request payload
- Check response status codes

**React DevTools:**
- Inspect component props/state
- View component hierarchy
- Check re-render causes

#### Common Issues

**Issue:** "Not authenticated" after login

**Debug:**
```javascript
// Check if token is stored
const auth = JSON.parse(localStorage.getItem('auth-storage'));
console.log('Token:', auth?.state?.token);

// Check API client headers
import { apiClient } from './api/client';
console.log('Headers:', apiClient.defaults.headers);
```

**Issue:** API calls failing with 401

**Debug:**
1. Check token in localStorage
2. Verify token is sent in Authorization header
3. Test token with curl:
```bash
curl http://localhost:8080/api/v1/auth/me \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Issue:** Component not re-rendering

**Debug:**
```typescript
// Add console.log to track renders
useEffect(() => {
  console.log('Component rendered', { tasks, loading });
}, [tasks, loading]);

// Check Zustand store
import { usePlanStore } from '@/stores/planStore';
console.log('Plan store:', usePlanStore.getState());
```

#### VS Code Debugging

**Launch Configuration (.vscode/launch.json):**

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Debug Frontend",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}/src"
    }
  ]
}
```

### Backend Debugging

#### IntelliJ IDEA

1. Set breakpoints in code
2. Run in Debug mode
3. Step through code
4. Evaluate expressions
5. View variable values

#### Logging

**Add debug logs:**

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class TaskService {

    public TaskDto createTask(String planId, LocalDate date, CreateTaskRequest request) {
        log.debug("Creating task for plan {} on date {}", planId, date);
        log.debug("Request: {}", request);

        try {
            // Implementation
            log.debug("Task created successfully: {}", result);
            return result;
        } catch (Exception e) {
            log.error("Failed to create task", e);
            throw e;
        }
    }
}
```

**Configure logging level (application.yml):**

```yaml
logging:
  level:
    com.example.weeklyplanner: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG
```

#### Common Issues

**Issue:** JWT token validation fails

**Debug:**
```java
@Component
@Slf4j
public class JwtTokenProvider {

    public boolean validateToken(String token) {
        log.debug("Validating token: {}", token);
        try {
            Claims claims = Jwts.parser()
                .setSigningKey(secretKey)
                .parseClaimsJws(token)
                .getBody();
            log.debug("Token claims: {}", claims);
            return true;
        } catch (Exception e) {
            log.error("Token validation failed", e);
            return false;
        }
    }
}
```

**Issue:** Database query not returning expected results

**Debug:**
```java
// Enable SQL logging
logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE

// Add query logging
@Query("SELECT t FROM Task t WHERE t.planId = :planId AND t.date = :date")
List<Task> findByPlanIdAndDate(@Param("planId") String planId, @Param("date") LocalDate date);

// In service
log.debug("Querying tasks with planId={}, date={}", planId, date);
List<Task> tasks = taskRepository.findByPlanIdAndDate(planId, date);
log.debug("Found {} tasks", tasks.size());
```

### API Debugging

#### Using curl with verbose output

```bash
# Show request/response headers
curl -v http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Save response to file
curl -o response.json http://localhost:8080/api/v1/plans/current \
  -H "Authorization: Bearer TOKEN"

# Show only headers
curl -I http://localhost:8080/api/v1/plans/current \
  -H "Authorization: Bearer TOKEN"
```

#### Using Postman

1. Import API collection
2. Set environment variables (BASE_URL, TOKEN)
3. Test endpoints
4. View response bodies
5. Check response times

#### Using test-api.js

```bash
# Run with debug output
DEBUG=true node test-api.js

# Test specific endpoint
node test-api.js --endpoint=/plans/current
```

---

## Performance Optimization

### Frontend Optimization

#### Code Splitting

```typescript
// Lazy load pages
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Planning = lazy(() => import('./pages/Planning'));
const Review = lazy(() => import('./pages/Review'));

// Route configuration
<Suspense fallback={<LoadingSpinner />}>
  <Routes>
    <Route path="/" element={<Dashboard />} />
    <Route path="/planning" element={<Planning />} />
    <Route path="/review" element={<Review />} />
  </Routes>
</Suspense>
```

#### React Performance

```typescript
// Memoize expensive computations
const statistics = useMemo(() => {
  return calculateStatistics(tasks);
}, [tasks]);

// Memoize callbacks
const handleStatusChange = useCallback((taskId: string, status: TaskStatus) => {
  updateTaskStatus(taskId, status);
}, [updateTaskStatus]);

// Memoize components
const TaskItem = memo(({ task, onStatusChange }) => {
  // Component implementation
}, (prevProps, nextProps) => {
  return prevProps.task.id === nextProps.task.id &&
         prevProps.task.status === nextProps.task.status;
});
```

#### Bundle Size Optimization

```bash
# Analyze bundle
npm run build -- --report

# Check for large dependencies
npx webpack-bundle-analyzer dist/stats.json

# Tree-shake unused code
import { format } from 'date-fns';  # ✅ Good
import * as dateFns from 'date-fns';  # ❌ Bad
```

#### Image Optimization

```typescript
// Use appropriate image formats
// WebP for photos, SVG for icons

// Lazy load images
<img
  src={imageUrl}
  loading="lazy"
  alt="Description"
/>

// Use srcset for responsive images
<img
  srcSet={`${image1x} 1x, ${image2x} 2x`}
  src={image1x}
  alt="Description"
/>
```

### Backend Optimization

#### Database Optimization

```java
// Use pagination
@Query("SELECT t FROM Task t WHERE t.planId = :planId")
Page<Task> findByPlanId(@Param("planId") String planId, Pageable pageable);

// Fetch associations efficiently
@EntityGraph(attributePaths = {"tasks"})
WeeklyPlan findById(String id);

// Use database indexes
@Table(name = "tasks", indexes = {
    @Index(name = "idx_plan_date", columnList = "plan_id, date"),
    @Index(name = "idx_user_id", columnList = "user_id")
})
```

#### Caching

```java
// Enable caching
@EnableCaching
@Configuration
public class CacheConfig { }

// Cache frequently accessed data
@Cacheable(value = "plans", key = "#planId")
public WeeklyPlan getPlan(String planId) {
    return planRepository.findById(planId)
        .orElseThrow(() -> new PlanNotFoundException(planId));
}

// Clear cache on updates
@CacheEvict(value = "plans", key = "#planId")
public WeeklyPlan updatePlan(String planId, UpdatePlanRequest request) {
    // Update logic
}
```

#### Query Optimization

```java
// Bad: N+1 query problem
List<WeeklyPlan> plans = planRepository.findAll();
plans.forEach(plan -> {
    List<Task> tasks = taskRepository.findByPlanId(plan.getId());  // N queries
});

// Good: Join fetch
@Query("SELECT p FROM WeeklyPlan p LEFT JOIN FETCH p.tasks WHERE p.id = :planId")
WeeklyPlan findByIdWithTasks(@Param("planId") String planId);
```

#### API Response Optimization

```java
// Use projection for partial data
public interface TaskSummary {
    String getId();
    String getTitle();
    TaskStatus getStatus();
}

@Query("SELECT t FROM Task t WHERE t.planId = :planId")
List<TaskSummary> findSummaryByPlanId(@Param("planId") String planId);
```

### Network Optimization

#### API Batching

```typescript
// Bad: Multiple requests
const plan = await planApi.getCurrent();
const tasks = await taskApi.list(plan.id);
const notifications = await notificationApi.list();

// Good: Parallel requests
const [plan, notifications] = await Promise.all([
  planApi.getCurrent(),
  notificationApi.list()
]);
const tasks = await taskApi.list(plan.id);
```

#### Request Debouncing

```typescript
// Debounce search input
const debouncedSearch = useDebouncedCallback((query: string) => {
  searchTasks(query);
}, 300);

<input onChange={(e) => debouncedSearch(e.target.value)} />
```

#### Response Compression

```java
// Enable gzip compression (application.yml)
server:
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/xml,text/plain
```

---

## Troubleshooting

### Common Problems

#### Submodule Issues

**Problem:** Submodule shows as modified after pull

```bash
# Check submodule status
git submodule status

# Reset to parent's reference
git submodule update --init

# Or update to latest
git submodule update --remote
```

**Problem:** Submodule directory is empty

```bash
git submodule init
git submodule update
```

**Problem:** Can't commit in submodule

```bash
cd docs/
git checkout main  # Must be on a branch, not detached HEAD
git pull
# Make changes
git commit -m "docs: update"
git push
cd ..
```

#### Build Issues

**Problem:** TypeScript errors after dependency update

```bash
rm -rf node_modules package-lock.json
npm install
npm run type-check
```

**Problem:** Backend build fails with dependency conflicts

```bash
./mvnw dependency:tree  # Check dependency tree
./mvnw clean install -U  # Force update dependencies
```

#### Runtime Issues

**Problem:** CORS errors in browser

**Solution:** Check backend CORS configuration:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/v1/**")
            .allowedOrigins("http://localhost:3000")
            .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
            .allowedHeaders("*")
            .allowCredentials(true);
    }
}
```

**Problem:** 401 Unauthorized after login

**Debug steps:**
1. Check token in localStorage
2. Verify token format in Authorization header
3. Test token with curl
4. Check backend token validation
5. Verify JWT secret matches

**Problem:** Tasks not appearing

**Debug:**
1. Check network tab for API calls
2. Verify response data format
3. Check Zustand store state
4. Verify component is subscribed to store
5. Check for console errors

### Getting Help

**When you're stuck:**

1. **Check documentation**
   - README.md
   - API contract
   - Integration guide

2. **Search codebase**
   ```bash
   grep -r "searchTerm" src/
   ```

3. **Check git history**
   ```bash
   git log --all --grep="related feature"
   git blame file.ts
   ```

4. **Review tests**
   - Look for similar test cases
   - Run tests to see what's expected

5. **Ask for help**
   - Provide error messages
   - Share relevant code
   - Describe what you tried

---

## Quick Reference

### Essential Commands

```bash
# Start development
git pull && git submodule update --remote
npm install  # or ./mvnw install
npm run dev  # or ./mvnw spring-boot:run

# Update documentation
cd docs/
git checkout main && git pull
# Make changes
git commit -m "docs: update" && git push
cd .. && git add docs && git commit -m "chore: update docs submodule"

# Test
npm run test  # Frontend
./mvnw test  # Backend
node test-api.js  # API

# Commit
git add .
git commit -m "feat(scope): description"
git push
```

### File Locations

```
Frontend:
- API clients: src/api/
- Components: src/components/
- Pages: src/pages/
- Types: src/types/
- Stores: src/stores/

Backend:
- Controllers: src/main/java/.../controller/
- Services: src/main/java/.../service/
- Repositories: src/main/java/.../repository/
- DTOs: src/main/java/.../dto/
- Entities: src/main/java/.../entity/

Docs:
- API: docs/api-contract.md
- Domain: docs/domain-model.md
- Business: docs/business-rules.md
- UI: docs/ui-spec.md
```

### Port Numbers

- Frontend Dev: http://localhost:3000
- Backend API: http://localhost:8080
- H2 Console: http://localhost:8080/h2-console (if enabled)

---

**Remember:** This is a living document. Update it as workflows evolve and new patterns emerge. If you find something unclear or missing, please improve it for the next developer!
