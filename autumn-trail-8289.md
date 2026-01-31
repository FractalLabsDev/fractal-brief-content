# talkwise-promptsmith PR #1 Review (v0.1)

**+61,490 / -3 lines | 62 files | 12 commits | ⚠️ Merge blocked**

This is the core Promptsmith microservice - the foundation for all the other PRs. Comprehensive review:

## Architecture

Clean, well-structured Express/TypeScript service following established TalkWise patterns:

```
src/
├── controllers/     # HTTP handlers (prompts, environments, runs, workflows)
├── models/          # Sequelize models (Prompt, PromptVersion, PromptRun, Environment, Workflow)
├── services/        # Business logic (execution, oracle integration, templates)
├── routes/          # Route definitions
├── middleware/      # Error handling, rate limiting, security
└── types/           # TypeScript interfaces
```

## Strengths

1. **Versioning** - Built-in prompt versioning with rollback support
2. **Environment Support** - Prompts can behave differently per environment
3. **Cost Tracking** - Calculates estimated cost based on model pricing from Oracle
4. **Run Logging** - Every execution creates a PromptRun record with timing/tokens/cost
5. **Import Script** - `import-pi-prompts.ts` for migrating from Wordware (47,187 lines of prompt data!)
6. **Workflow Support** - Foundation for chaining prompts together (future use)
7. **Setup Script** - `npm run setup` automates dev environment setup

## Key Implementation Details

**Execution Flow** (`execution.service.ts`):
1. Lookup prompt by ID or slug
2. Get/create environment
3. Create PromptRun record
4. Call Oracle with content + inputs
5. Record usage metrics (tokens, cost, duration)
6. Return result

**Slug-based lookup**: Prompts can be accessed by UUID or human-readable slug - nice for API consumers.

**App scoping**: `appName` parameter prepared for future per-app prompt namespacing.

## Minor Suggestions

1. **Error handling in calculateCost**: Currently swallows errors and returns 0. Consider logging at `warn` level consistently.

2. **Magic numbers**: Some defaults like `temperature: 0.7` could be constants.

3. **Rate limiting config**: 10,000 requests per 15 min is very generous - verify this is intentional for dev.

4. **data/ directory**: The 47K line JSON file (`practice-interviews-structure.json`) is big. Consider:
   - Adding to `.gitignore` after first import
   - Or moving to a separate data repo/S3

## Summary

Solid foundation for a prompt management system. The architecture is clean, the code is well-organized, and it follows the established patterns from other TalkWise services. The Wordware import data is massive but necessary for the migration.

Ready to merge pending CI/approval.
