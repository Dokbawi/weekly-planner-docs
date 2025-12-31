# Weekly Planner - Documentation Index

**Last Updated:** 2025-12-29

Welcome to the Weekly Planner documentation! This repository serves as a git submodule shared between the frontend and backend repositories, ensuring consistency across all project components.

---

## Overview

Weekly Planner is a productivity-focused web application that supports a complete planning cycle:
- **Weekly Planning**: Create structured plans for the entire week
- **Daily Execution**: Manage and track daily tasks
- **Change Tracking**: Monitor how plans evolve over time
- **Weekly Review**: Analyze completion rates and learn from changes

The documentation is centralized in this submodule to maintain a single source of truth for:
- Domain models and business logic
- API contracts and specifications
- UI/UX specifications
- Integration guidelines

---

## Documentation Structure

### This Repository (docs submodule)
Contains **shared specifications** used by both frontend and backend:
- Domain models
- API contracts
- Business rules
- UI specifications
- Integration guides

### Main Repositories
Each repository has its own `CLAUDE.md` for implementation-specific details:
- **Frontend CLAUDE.md**: React components, state management, UI implementation
- **Backend CLAUDE.md**: Spring Boot services, database schemas, API implementation
- **Docs CLAUDE.md**: Documentation maintenance rules

---

## Quick Links

### Core Specifications

#### [Domain Model](./domain-model.md)
**What it covers:**
- Entity definitions (User, WeeklyPlan, Task, ChangeLog, etc.)
- Data types and field specifications
- Entity relationships
- Status enums and their meanings

**When to use:**
- Understanding core business entities
- Implementing database schemas
- Creating TypeScript types or Java entities
- Clarifying data structure questions

---

#### [API Contract](./api-contract.md)
**What it covers:**
- Complete REST API specification
- Request/response formats
- Authentication requirements
- Endpoint paths and HTTP methods
- Query parameters and request bodies

**When to use:**
- Implementing backend endpoints
- Writing frontend API clients
- Testing API integration
- Debugging API calls

**Status:** Updated 2025-12-29
- Some endpoints marked as optional implementation
- `/plans/current`, `/today`, `/reviews/current` have workarounds in frontend

---

#### [Business Rules](./business-rules.md)
**What it covers:**
- Weekly plan lifecycle (DRAFT → CONFIRMED → COMPLETED)
- Task state transitions and constraints
- Change tracking rules (when ChangeLog is created)
- Review generation logic
- Validation requirements

**When to use:**
- Implementing business logic
- Understanding workflow constraints
- Validating user actions
- Writing automated tests

---

#### [UI Specification](./ui-spec.md)
**What it covers:**
- Page layouts and navigation structure
- Component specifications
- User interactions and flows
- Responsive design requirements
- Visual hierarchy and information architecture

**When to use:**
- Designing frontend components
- Understanding user workflows
- Planning UX improvements
- Ensuring consistent UI patterns

---

### Integration & Development

#### [Backend Integration Guide](./backend-integration-guide.md)
**What it covers:**
- Frontend's API integration requirements
- Implementation status of endpoints
- Required vs. optional endpoints
- Response format requirements
- Common integration issues and solutions
- Testing tools for API validation

**Target audience:** Backend developers implementing APIs

**When to use:**
- Starting backend implementation
- Debugging frontend-backend integration
- Understanding what the frontend expects
- Testing your API endpoints

---

#### [Development Workflow](./development-workflow.md)
**What it covers:**
- Git submodule management
- Development environment setup
- API development best practices
- Commit message conventions
- Testing strategies
- Documentation standards
- Common tasks and commands

**Target audience:** All developers (Frontend, Backend, Full-Stack)

**When to use:**
- Setting up your development environment
- Learning project conventions
- Managing submodule updates
- Following best practices
- Troubleshooting common issues

---

#### [CLAUDE.md](./CLAUDE.md)
**What it covers:**
- Documentation repository structure
- Documentation update rules
- Submodule synchronization guidelines

**Target audience:** AI assistants and documentation maintainers

---

## For Different Roles

### Frontend Developers

**Essential reading (in order):**
1. [Domain Model](./domain-model.md) - Understand data structures
2. [API Contract](./api-contract.md) - Learn API endpoints
3. [UI Specification](./ui-spec.md) - Understand UI requirements
4. [Business Rules](./business-rules.md) - Implement validation logic
5. [Backend Integration Guide](./backend-integration-guide.md) - Know API status
6. [Development Workflow](./development-workflow.md) - Follow best practices

**Your main repository CLAUDE.md:** `weekly-planner-frontend/CLAUDE.md`
- React components and structure
- State management (Zustand stores)
- API client implementation
- Component library (shadcn/ui)

---

### Backend Developers

**Essential reading (in order):**
1. [Domain Model](./domain-model.md) - Design database schemas
2. [Business Rules](./business-rules.md) - Implement core logic
3. [API Contract](./api-contract.md) - Implement endpoints
4. [Backend Integration Guide](./backend-integration-guide.md) - Meet frontend requirements
5. [Development Workflow](./development-workflow.md) - Follow conventions

**Your main repository CLAUDE.md:** `weekly-planner-backend/CLAUDE.md`
- Spring Boot architecture
- MongoDB repositories
- Service layer implementation
- Security configuration

---

### Full-Stack Developers

**Recommended reading order:**
1. [Domain Model](./domain-model.md) - Core understanding
2. [Business Rules](./business-rules.md) - Business logic
3. [API Contract](./api-contract.md) - Integration layer
4. [Backend Integration Guide](./backend-integration-guide.md) - Current status
5. [UI Specification](./ui-spec.md) - User experience
6. [Development Workflow](./development-workflow.md) - Best practices

Then refer to both frontend and backend CLAUDE.md files for implementation details.

---

### AI Assistants (Claude, ChatGPT, etc.)

**Priority documents:**
1. **[CLAUDE.md](./CLAUDE.md)** - Start here for context
2. **[Development Workflow](./development-workflow.md)** - Understand conventions
3. **[Backend Integration Guide](./backend-integration-guide.md)** - Current implementation status
4. **Domain Model → API Contract → Business Rules** - For detailed specifications

**When working on tasks:**
- Frontend tasks: Refer to frontend CLAUDE.md + API Contract
- Backend tasks: Refer to backend CLAUDE.md + Backend Integration Guide
- Documentation tasks: Refer to Documentation Update Guidelines below

---

## Documentation Update Guidelines

### When to Update Which Document

#### Domain Model Changes
**Update:** `domain-model.md`
**Then cascade to:**
- Backend: Update MongoDB models
- Frontend: Update TypeScript types
- API Contract: Update request/response schemas

**Example:** Adding a new field to Task entity

---

#### API Changes
**Update:** `api-contract.md`
**Then cascade to:**
- Backend: Implement new endpoints
- Frontend: Update API client
- Backend Integration Guide: Update implementation status

**Example:** Adding a new endpoint or changing response format

---

#### Business Logic Changes
**Update:** `business-rules.md`
**Then cascade to:**
- Backend: Update service layer validation
- Frontend: Update UI validation
- UI Spec: Update user flow if needed

**Example:** Changing task state transition rules

---

#### UI/UX Changes
**Update:** `ui-spec.md`
**Then cascade to:**
- Frontend: Update components
- API Contract: Add new endpoints if needed
- Domain Model: Add new fields if needed

**Example:** Adding a new page or changing user workflow

---

### How to Keep Docs in Sync

#### 1. Update Sequence
```bash
# Step 1: Update documentation in docs repository
cd docs/
git checkout main
git pull
# Edit relevant .md files
git add .
git commit -m "docs: update API contract for new task fields"
git push

# Step 2: Update submodule reference in frontend
cd ../weekly-planner-frontend/
git submodule update --remote docs
git add docs
git commit -m "chore: update docs submodule reference"
git push

# Step 3: Update submodule reference in backend
cd ../weekly-planner-backend/
git submodule update --remote docs
git add docs
git commit -m "chore: update docs submodule reference"
git push
```

---

#### 2. Verify Consistency
Before committing changes, verify:
- [ ] Domain model matches both frontend types and backend entities
- [ ] API contract matches backend implementation
- [ ] Business rules are enforced in both frontend and backend
- [ ] UI spec matches current frontend implementation

---

#### 3. Documentation Review Checklist
When updating documentation:
- [ ] Clear and concise language
- [ ] Examples provided where helpful
- [ ] Cross-references to related documents
- [ ] Version/date updated
- [ ] Breaking changes clearly marked
- [ ] Implementation status noted (if applicable)

---

#### 4. Breaking Changes
When making breaking changes:
1. Mark clearly in the documentation
2. Update API Contract with version notes
3. Update Backend Integration Guide
4. Coordinate implementation across repositories
5. Update both frontend and backend CLAUDE.md files

---

## Getting Help

### Questions About...

**Domain Models & Entities**
→ Read [Domain Model](./domain-model.md)
→ Check entity relationships section

**API Endpoints**
→ Read [API Contract](./api-contract.md)
→ Check [Backend Integration Guide](./backend-integration-guide.md) for status

**Business Logic & Rules**
→ Read [Business Rules](./business-rules.md)
→ Look for specific workflows (e.g., task lifecycle)

**UI/UX Flows**
→ Read [UI Specification](./ui-spec.md)
→ Check page-by-page breakdown

**Development Setup**
→ Read [Development Workflow](./development-workflow.md)
→ Check troubleshooting section

**Integration Issues**
→ Read [Backend Integration Guide](./backend-integration-guide.md)
→ Check common integration issues section

---

## Contributing

### Adding New Documentation
1. Create the document with clear sections
2. Add entry to this README.md
3. Update [CLAUDE.md](./CLAUDE.md) if it affects structure
4. Cross-reference from related documents
5. Update submodules in main repositories

### Improving Existing Documentation
1. Maintain consistency with existing format
2. Update "Last Updated" date
3. Add version notes if breaking changes
4. Verify cross-references still valid
5. Update this README if scope changes

---

## Document Status

| Document | Status | Last Updated | Completeness |
|----------|--------|--------------|--------------|
| Domain Model | ✅ Stable | 2024-12-21 | Complete |
| API Contract | ✅ Stable | 2025-12-29 | Complete* |
| Business Rules | ✅ Stable | 2024-12-21 | Complete |
| UI Specification | ✅ Stable | 2024-12-21 | Complete |
| Backend Integration Guide | ✅ Stable | 2025-12-29 | Complete |
| Development Workflow | ✅ Stable | 2025-12-29 | Complete |
| CLAUDE.md | ✅ Stable | 2024-12-21 | Complete |

\* Some optional endpoints documented but not implemented in backend

---

## Version History

### v1.3.0 (2025-12-29)
- Added comprehensive README.md index
- Updated Backend Integration Guide with testing tools
- Enhanced Development Workflow with best practices

### v1.2.0 (2025-12-29)
- Added Backend Integration Guide
- Added Development Workflow Guide
- Updated API Contract with implementation status notes

### v1.1.0 (2024-12-28)
- Frontend API integration adjustments
- Documented optional endpoints and workarounds

### v1.0.0 (2024-12-21)
- Initial documentation structure
- Core specifications complete
- Domain Model, API Contract, Business Rules, UI Spec

---

**Need help? Have questions?** Check the appropriate documentation above or refer to the main repository CLAUDE.md files for implementation-specific details.

---

## Navigation

### Main Documentation Files
- [API Contract](./api-contract.md)
- [Domain Model](./domain-model.md)
- [Business Rules](./business-rules.md)
- [UI Specification](./ui-spec.md)

### Development Guides
- [Backend Integration Guide](./backend-integration-guide.md)
- [Development Workflow](./development-workflow.md)
- [CLAUDE.md](./CLAUDE.md)

---

**Last Updated:** 2025-12-31
