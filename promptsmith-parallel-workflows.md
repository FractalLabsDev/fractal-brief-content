# Implementing Parallel Workflow Execution in Promptsmith

**Date:** 2026-02-03
**Author:** Ruk
**Status:** Proposal - Pending Review

---

## Executive Summary

This brief outlines how to implement the parallel workflow execution functionality from `talkwise-backend` in the new `talkwise-promptsmith` system. The goal is a **hybrid execution engine** that supports:

1. **Parallel execution** - Multiple prompts run concurrently when they have no dependencies
2. **Sequential execution** - Prompts run in order when they depend on previous outputs
3. **Hybrid patterns** - Combinations where parallel groups feed into sequential aggregation steps

The target use case is Practice Interviews CFAS feedback, where four critique prompts (Clarify, Framework, Assumptions, Solution) can run in parallel, but Summary and Score prompts require all four outputs as input.

---

## Current State Analysis

### Legacy System (`talkwise-backend`)

**Location:** `talkwise-backend/src/services/internal/workflow.service.ts`

The legacy system uses a **sections-based** workflow definition with explicit type declarations:

```typescript
// Section types
type: "sequential" | "parallel" | "branching"

// Workflow structure
{
  sections: [
    {
      type: "parallel",
      prompts: [...],    // All execute concurrently
      followUp: {...}    // Optional aggregation prompt after parallel completes
    },
    {
      type: "sequential",
      prompts: [...]     // Execute in order, each sees previous outputs
    }
  ]
}
```

**Key Implementation Details:**

1. **Parallel Execution** (lines 138-172):
   - Uses `Promise.all()` to wait for all prompts
   - Includes 1-second stagger to avoid rate limiting
   - Stores each result in `context[prompt.name]`
   - Optional `followUp` prompt runs after all parallel prompts complete

2. **Sequential Execution** (lines 112-135):
   - Simple for-loop through prompts
   - Each prompt result stored in context for next prompt
   - `{{context.variableName}}` placeholder substitution

3. **Context Passing:**
   - Global `this.context` object accumulates all prompt outputs
   - Each prompt can reference any previous prompt's output
   - Streaming vs non-streaming handled per-prompt

### New System (`talkwise-promptsmith`)

**Location:** `talkwise-promptsmith/src/services/workflow-execution.service.ts`

The current promptsmith system uses a **dependency-graph** approach:

```typescript
interface WorkflowStep {
  id: string;
  promptId: string;
  outputKey: string;
  dependsOn: string[];  // Step IDs this step depends on
}
```

**Current Implementation** (lines 62-126):

```typescript
while (completedSteps.size < steps.length) {
  // Find steps ready to execute (all dependencies completed)
  const readySteps = steps.filter(step => {
    if (completedSteps.has(step.id)) return false;
    return (step.dependsOn || []).every(depId => completedSteps.has(depId));
  });

  // Execute ready steps in parallel
  await Promise.all(readySteps.map(async step => {
    const result = await executePrompt(step.promptId, {...inputs, ...stepOutputs});
    stepOutputs[step.outputKey] = result.result;
    completedSteps.add(step.id);
  }));
}
```

**This already supports parallel execution!** Steps with no dependencies (or whose dependencies are all satisfied) run concurrently via `Promise.all()`.

---

## Architecture Comparison

| Aspect | Legacy (talkwise-backend) | New (promptsmith) |
|--------|---------------------------|-------------------|
| Execution Model | Explicit sections (parallel/sequential) | Implicit via dependency graph |
| Parallel Control | Section-level (`type: "parallel"`) | Automatic based on `dependsOn` |
| Streaming | Per-prompt via `streamResult` | Not yet implemented |
| Context Passing | `{{context.varName}}` placeholders | Input key mapping |
| Rate Limiting | 1-second stagger in parallel | None currently |
| Aggregation | `followUp` prompt in parallel sections | Any step can depend on multiple |

---

## Proposed Implementation

### Design Philosophy

**Retain the dependency-graph model** in promptsmith as it's more elegant and flexible than explicit section types. The current model already handles parallel vs sequential implicitly - we just need to enhance it with:

1. **Rate limiting** for parallel executions
2. **Streaming support** for real-time feedback
3. **Template variable substitution** for prompt content
4. **Input mapping** to translate output keys to input variables
5. **Output transformation** for score extraction, etc.

### Schema Changes

#### 1. Enhanced WorkflowStep Interface

```typescript
// File: src/types/workflow.types.ts

export interface WorkflowStep {
  id: string;
  promptId: string;              // UUID or slug of the prompt
  outputKey: string;             // Key to store result under
  dependsOn: string[];           // Step IDs this depends on

  // NEW: Input mapping
  inputMapping?: Record<string, string>;  // Maps step outputs to prompt inputs
                                          // e.g., { "clarify_feedback": "clarify.output" }

  // NEW: Execution options
  options?: {
    stream?: boolean;            // Enable streaming response
    timeout?: number;            // Custom timeout (ms)
    retryCount?: number;         // Number of retries on failure
    outputTransform?: string;    // Transform function name (e.g., "extractScores")
  };
}

export interface WorkflowDefinition {
  id: string;
  slug: string;
  name: string;
  description?: string;
  steps: WorkflowStep[];

  // NEW: Global workflow options
  options?: {
    parallelBatchDelay?: number;  // Delay between parallel batch starts (ms)
    maxParallelism?: number;      // Max concurrent executions
    defaultModel?: string;        // Override model for all prompts
  };
}
```

#### 2. Enhanced Workflow Model

```typescript
// File: src/models/Workflow.ts (additions)

@Default({})
@Column({
  type: DataType.JSONB,
  allowNull: false,
})
options!: {
  parallelBatchDelay?: number;
  maxParallelism?: number;
  defaultModel?: string;
};

@Column({
  type: DataType.STRING,
  allowNull: true,
})
appName?: string;  // For app-scoped workflows (practice-interviews, talkwise, etc.)
```

#### 3. New Migration

```javascript
// File: src/migrations/YYYYMMDD-add-workflow-options.js

module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn('workflows', 'options', {
      type: Sequelize.JSONB,
      allowNull: false,
      defaultValue: {},
    });

    await queryInterface.addColumn('workflows', 'app_name', {
      type: Sequelize.STRING,
      allowNull: true,
    });
  },

  down: async (queryInterface) => {
    await queryInterface.removeColumn('workflows', 'options');
    await queryInterface.removeColumn('workflows', 'app_name');
  },
};
```

### Service Changes

#### 1. Enhanced Workflow Execution Service

```typescript
// File: src/services/workflow-execution.service.ts

import Workflow, { WorkflowStep } from '@models/Workflow';
import Prompt from '@models/Prompt';
import { executePrompt, executePromptStreaming } from './execution.service';
import { applyOutputTransform } from './transform.service';
import logger from '@utils/logger';
import pLimit from 'p-limit';  // For concurrency control

export interface WorkflowExecutionOptions {
  stream?: boolean;
  onStepStart?: (stepId: string, promptId: string) => void;
  onStepComplete?: (stepId: string, output: string) => void;
  onStepStream?: (stepId: string, chunk: string) => void;
}

export interface WorkflowExecutionResult {
  workflowId: string;
  stepResults: Record<string, StepResult>;
  finalOutputs: Record<string, string>;
  totalUsage: UsageSummary;
}

interface StepResult {
  stepId: string;
  promptId: string;
  output: string;
  transformedOutput?: any;
  usage: {
    inputTokens: number;
    outputTokens: number;
    estimatedCost: number;
    durationMs: number;
  };
}

interface UsageSummary {
  inputTokens: number;
  outputTokens: number;
  estimatedCost: number;
  totalDurationMs: number;
  parallelEfficiency: number;  // Ratio of sequential time to actual time
}

/**
 * Execute a workflow using dependency-based parallel/sequential execution
 */
export async function executeWorkflow(
  workflowIdOrSlug: string,
  inputs: Record<string, string>,
  environmentId?: string,
  userId?: string,
  options: WorkflowExecutionOptions = {}
): Promise<WorkflowExecutionResult> {
  const workflow = await Workflow.findOne({
    where: {
      [Op.or]: [{ id: workflowIdOrSlug }, { slug: workflowIdOrSlug }],
    },
  });

  if (!workflow) {
    throw new Error(`Workflow not found: ${workflowIdOrSlug}`);
  }

  const steps = workflow.steps;
  if (!steps || steps.length === 0) {
    throw new Error('Workflow has no steps');
  }

  // Execution state
  const stepMap = new Map<string, WorkflowStep>();
  const completedSteps = new Set<string>();
  const stepResults: Record<string, StepResult> = {};
  const stepOutputs: Record<string, string> = { ...inputs };  // Start with initial inputs

  for (const step of steps) {
    stepMap.set(step.id, step);
  }

  // Usage tracking
  let totalInputTokens = 0;
  let totalOutputTokens = 0;
  let totalCost = 0;
  let sequentialTimeEstimate = 0;  // For efficiency calculation
  const startTime = Date.now();

  // Concurrency control
  const workflowOptions = workflow.options || {};
  const maxParallelism = workflowOptions.maxParallelism || 10;
  const batchDelay = workflowOptions.parallelBatchDelay || 100;  // 100ms default
  const limit = pLimit(maxParallelism);

  // Execute in topological order with parallel batching
  while (completedSteps.size < steps.length) {
    // Find steps ready to execute
    const readySteps = steps.filter((step) => {
      if (completedSteps.has(step.id)) return false;
      const deps = step.dependsOn || [];
      return deps.every((depId) => completedSteps.has(depId));
    });

    if (readySteps.length === 0) {
      const remaining = steps.filter(s => !completedSteps.has(s.id));
      throw new Error(
        `Workflow stalled. Remaining steps: ${remaining.map(s => s.id).join(', ')}. ` +
        `Check for circular dependencies or missing steps.`
      );
    }

    logger.info(
      {
        workflowId: workflow.id,
        batchSize: readySteps.length,
        completedCount: completedSteps.size,
        totalSteps: steps.length
      },
      `Executing parallel batch of ${readySteps.length} steps`
    );

    // Execute ready steps with controlled parallelism
    const batchPromises = readySteps.map((step, index) =>
      limit(async () => {
        // Stagger starts to avoid rate limiting
        if (index > 0 && batchDelay > 0) {
          await sleep(index * batchDelay);
        }

        const stepStartTime = Date.now();

        // Notify step start
        options.onStepStart?.(step.id, step.promptId);

        // Build inputs for this step
        const stepInputs = buildStepInputs(step, stepOutputs);

        // Execute prompt
        const useStreaming = options.stream && step.options?.stream !== false;

        let result;
        if (useStreaming && options.onStepStream) {
          result = await executePromptStreaming(
            step.promptId,
            stepInputs,
            environmentId,
            userId,
            workflow.appName,
            (chunk) => options.onStepStream!(step.id, chunk)
          );
        } else {
          result = await executePrompt(
            step.promptId,
            stepInputs,
            environmentId,
            userId,
            workflow.appName
          );
        }

        const stepDuration = Date.now() - stepStartTime;
        sequentialTimeEstimate += stepDuration;

        // Apply output transformation if specified
        let transformedOutput = result.result;
        if (step.options?.outputTransform) {
          transformedOutput = applyOutputTransform(
            result.result,
            step.options.outputTransform
          );
        }

        // Store result
        stepResults[step.id] = {
          stepId: step.id,
          promptId: step.promptId,
          output: result.result,
          transformedOutput: transformedOutput !== result.result ? transformedOutput : undefined,
          usage: result.usage,
        };

        // Store output for downstream steps
        stepOutputs[step.outputKey] = result.result;
        if (transformedOutput !== result.result) {
          stepOutputs[`${step.outputKey}_transformed`] =
            typeof transformedOutput === 'string'
              ? transformedOutput
              : JSON.stringify(transformedOutput);
        }

        // Update totals
        totalInputTokens += result.usage.inputTokens;
        totalOutputTokens += result.usage.outputTokens;
        totalCost += result.usage.estimatedCost;

        completedSteps.add(step.id);

        // Notify step complete
        options.onStepComplete?.(step.id, result.result);

        logger.debug(
          {
            workflowId: workflow.id,
            stepId: step.id,
            outputKey: step.outputKey,
            durationMs: stepDuration,
          },
          'Workflow step completed'
        );
      })
    );

    // Wait for all parallel steps to complete
    await Promise.all(batchPromises);
  }

  const totalDurationMs = Date.now() - startTime;
  const parallelEfficiency = sequentialTimeEstimate / totalDurationMs;

  logger.info(
    {
      workflowId: workflow.id,
      totalDurationMs,
      sequentialTimeEstimate,
      parallelEfficiency: parallelEfficiency.toFixed(2),
      stepsCompleted: completedSteps.size,
    },
    'Workflow execution completed'
  );

  return {
    workflowId: workflow.id,
    stepResults,
    finalOutputs: stepOutputs,
    totalUsage: {
      inputTokens: totalInputTokens,
      outputTokens: totalOutputTokens,
      estimatedCost: totalCost,
      totalDurationMs,
      parallelEfficiency,
    },
  };
}

/**
 * Build inputs for a step by mapping outputs from dependencies
 */
function buildStepInputs(
  step: WorkflowStep,
  stepOutputs: Record<string, string>
): Record<string, string> {
  const inputs: Record<string, string> = {};

  // If inputMapping is defined, use it
  if (step.inputMapping) {
    for (const [inputKey, outputPath] of Object.entries(step.inputMapping)) {
      // outputPath can be simple key or dot-notation (e.g., "clarify.output")
      const value = resolveOutputPath(outputPath, stepOutputs);
      if (value !== undefined) {
        inputs[inputKey] = value;
      }
    }
  } else {
    // Default: pass all available outputs
    Object.assign(inputs, stepOutputs);
  }

  return inputs;
}

/**
 * Resolve a dot-notation path to a value in stepOutputs
 */
function resolveOutputPath(
  path: string,
  stepOutputs: Record<string, string>
): string | undefined {
  // Simple case: direct key
  if (stepOutputs[path] !== undefined) {
    return stepOutputs[path];
  }

  // Dot notation: try to resolve
  const parts = path.split('.');
  if (parts.length === 1) {
    return stepOutputs[parts[0]];
  }

  // For now, just try the first part
  return stepOutputs[parts[0]];
}

function sleep(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

#### 2. Output Transform Service

```typescript
// File: src/services/transform.service.ts

/**
 * Apply output transformations to prompt results
 */
export function applyOutputTransform(output: string, transformName: string): any {
  const transforms: Record<string, (output: string) => any> = {
    extractScores: extractScores,
    extractQuestions: extractQuestions,
    extractWordwareScores: extractWordwareScores,
    parseJson: parseJsonSafe,
    extractMarkdownList: extractMarkdownList,
  };

  const transform = transforms[transformName];
  if (!transform) {
    throw new Error(`Unknown output transform: ${transformName}`);
  }

  return transform(output);
}

function extractScores(output: string): Record<string, number> {
  // Match patterns like "Score: 8/10" or "**Clarity**: 7"
  const scores: Record<string, number> = {};
  const patterns = [
    /(\w+):\s*(\d+)(?:\/10)?/gi,
    /\*\*(\w+)\*\*:\s*(\d+)/gi,
  ];

  for (const pattern of patterns) {
    let match;
    while ((match = pattern.exec(output)) !== null) {
      scores[match[1].toLowerCase()] = parseInt(match[2], 10);
    }
  }

  return scores;
}

function extractQuestions(output: string): string[] {
  // Extract numbered questions
  const questions: string[] = [];
  const lines = output.split('\n');

  for (const line of lines) {
    const match = line.match(/^\d+[\.\)]\s*(.+)/);
    if (match) {
      questions.push(match[1].trim());
    }
  }

  return questions;
}

function extractWordwareScores(output: string): Record<string, any> {
  // Wordware returns JSON-like structured scores
  return parseJsonSafe(output);
}

function parseJsonSafe(output: string): any {
  try {
    // Try to find JSON in the output
    const jsonMatch = output.match(/\{[\s\S]*\}/);
    if (jsonMatch) {
      return JSON.parse(jsonMatch[0]);
    }
    return output;
  } catch {
    return output;
  }
}

function extractMarkdownList(output: string): string[] {
  const items: string[] = [];
  const lines = output.split('\n');

  for (const line of lines) {
    const match = line.match(/^[-*]\s+(.+)/);
    if (match) {
      items.push(match[1].trim());
    }
  }

  return items;
}
```

#### 3. Enhanced Execution Service with Streaming

```typescript
// File: src/services/execution.service.ts (additions)

import { Readable } from 'stream';

/**
 * Execute a prompt with streaming response
 */
export async function executePromptStreaming(
  promptId: string,
  inputs: Record<string, string>,
  environmentId?: string,
  _userId?: string,
  appName?: string,
  onChunk?: (chunk: string) => void
): Promise<{
  result: string;
  runId: string;
  usage: {
    inputTokens: number;
    outputTokens: number;
    estimatedCost: number;
    durationMs: number;
  };
}> {
  const startTime = Date.now();

  const prompt = await promptService.getPrompt(promptId, appName);
  if (!prompt) {
    throw new Error(`Prompt not found: ${promptId}`);
  }

  const version = prompt.currentVersion || {
    content: prompt.content,
    modelId: prompt.modelId,
    temperature: prompt.temperature,
    inputs: prompt.inputs,
  };

  // Get environment (same as non-streaming)
  const environment = await getOrCreateEnvironment(environmentId);

  // Create run record
  const run = await PromptRun.create({
    promptId: prompt.id,
    promptVersionId: version instanceof PromptVersion ? version.id : prompt.id,
    environmentId: environment.id,
    inputs,
    status: 'running',
    durationMs: 0,
    inputTokens: 0,
    outputTokens: 0,
    estimatedCost: 0,
  });

  try {
    // Build final content
    let finalContent = version.content;
    if (Object.keys(inputs).length > 0) {
      const inputsSection = '\n\nInputs:\n' +
        Object.entries(inputs)
          .map(([key, value]) => `${key}: ${value}`)
          .join('\n');
      finalContent = version.content + inputsSection;
    }

    // Prepare messages
    const messages: ChatMessage[] = [
      { role: 'user', content: finalContent },
    ];

    // Call Oracle with streaming
    const stream = await oracleService.generateStream({
      messages,
      languageModel: version.modelId,
    });

    // Collect chunks
    let fullResult = '';
    for await (const chunk of stream) {
      fullResult += chunk;
      onChunk?.(chunk);
    }

    const durationMs = Date.now() - startTime;

    // Estimate tokens (actual usage may come from Oracle response)
    const estimatedInputTokens = Math.ceil(finalContent.length / 4);
    const estimatedOutputTokens = Math.ceil(fullResult.length / 4);
    const estimatedCost = await calculateCost(
      estimatedInputTokens,
      estimatedOutputTokens,
      version.modelId
    );

    // Update run record
    await run.update({
      status: 'completed',
      output: fullResult,
      durationMs,
      inputTokens: estimatedInputTokens,
      outputTokens: estimatedOutputTokens,
      estimatedCost,
    });

    return {
      result: fullResult,
      runId: run.id,
      usage: {
        inputTokens: estimatedInputTokens,
        outputTokens: estimatedOutputTokens,
        estimatedCost,
        durationMs,
      },
    };
  } catch (error: any) {
    const durationMs = Date.now() - startTime;
    await run.update({
      status: 'failed',
      error: error.message,
      durationMs,
    });
    throw error;
  }
}
```

### Example Workflow: CFAS Feedback

Here's how the CFAS (Clarify, Framework, Assumptions, Solution) workflow would be defined:

```json
{
  "slug": "cfas-feedback",
  "name": "CFAS Hypothetical Feedback",
  "description": "Feedback on hypothetical interview questions using CFAS framework",
  "appName": "practice-interviews",
  "options": {
    "parallelBatchDelay": 200,
    "maxParallelism": 4
  },
  "steps": [
    {
      "id": "clarify",
      "promptId": "hypothetical-clarify",
      "outputKey": "clarify_feedback",
      "dependsOn": [],
      "options": { "stream": true }
    },
    {
      "id": "framework",
      "promptId": "hypothetical-framework",
      "outputKey": "framework_feedback",
      "dependsOn": [],
      "options": { "stream": true }
    },
    {
      "id": "assumptions",
      "promptId": "hypothetical-assumptions",
      "outputKey": "assumptions_feedback",
      "dependsOn": [],
      "options": { "stream": true }
    },
    {
      "id": "solution",
      "promptId": "hypothetical-solution",
      "outputKey": "solution_feedback",
      "dependsOn": [],
      "options": { "stream": true }
    },
    {
      "id": "summary",
      "promptId": "hypothetical-summary",
      "outputKey": "summary",
      "dependsOn": ["clarify", "framework", "assumptions", "solution"],
      "inputMapping": {
        "clarify_feedback": "clarify_feedback",
        "framework_feedback": "framework_feedback",
        "assumptions_feedback": "assumptions_feedback",
        "solution_feedback": "solution_feedback"
      },
      "options": { "stream": true }
    },
    {
      "id": "score",
      "promptId": "hypothetical-score",
      "outputKey": "scores",
      "dependsOn": ["clarify", "framework", "assumptions", "solution"],
      "inputMapping": {
        "clarify_feedback": "clarify_feedback",
        "framework_feedback": "framework_feedback",
        "assumptions_feedback": "assumptions_feedback",
        "solution_feedback": "solution_feedback"
      },
      "options": {
        "stream": false,
        "outputTransform": "extractScores"
      }
    }
  ]
}
```

**Execution Flow:**

```
Time 0ms:   Start → [clarify, framework, assumptions, solution] (parallel)
Time ~3s:   All four complete → [summary, score] (parallel, both depend on all four)
Time ~5s:   Complete
```

Without parallelism, this would take ~12 seconds (6 prompts × ~2s each). With parallelism: ~5 seconds.

---

## Implementation Plan

### Phase 1: Core Infrastructure (PR #1)

**Files to create/modify:**

1. **New Migration**: `src/migrations/YYYYMMDD-add-workflow-options.js`
2. **Update Model**: `src/models/Workflow.ts` - Add `options` and `appName` columns
3. **Update Types**: `src/types/workflow.types.ts` - Enhanced interfaces
4. **New Service**: `src/services/transform.service.ts` - Output transformations
5. **Update Service**: `src/services/workflow-execution.service.ts` - Full rewrite with parallel batching

**Dependencies to add:**
- `p-limit` (concurrency control)

### Phase 2: Streaming Support (PR #2)

**Files to modify:**

1. **Update Service**: `src/services/execution.service.ts` - Add `executePromptStreaming`
2. **Update Service**: `src/services/oracle.service.ts` - Add `generateStream` method
3. **Update Controller**: `src/controllers/workflows.controller.ts` - SSE endpoint

### Phase 3: API & Integration (PR #3)

**Files to modify:**

1. **Update Routes**: `src/routes/workflows.routes.ts` - Streaming endpoint
2. **New Controller**: `src/controllers/workflow-execution.controller.ts` - Execution management
3. **Integration Tests**: Create test workflows and verify parallel execution

---

## Files to Change Summary

| File | Action | Description |
|------|--------|-------------|
| `src/types/workflow.types.ts` | Modify | Add `inputMapping`, `options` to WorkflowStep |
| `src/models/Workflow.ts` | Modify | Add `options`, `appName` columns |
| `src/services/workflow-execution.service.ts` | Rewrite | Full parallel execution with batching |
| `src/services/transform.service.ts` | Create | Output transformation functions |
| `src/services/execution.service.ts` | Modify | Add streaming support |
| `src/services/oracle.service.ts` | Modify | Add streaming generation |
| `src/migrations/YYYYMMDD-*.js` | Create | Schema changes |
| `package.json` | Modify | Add `p-limit` dependency |

---

## Risk Assessment

### Low Risk
- Type changes are additive (no breaking changes)
- New columns have defaults
- Existing workflows continue to work

### Medium Risk
- Parallel execution may hit rate limits on Oracle/LLM providers
- Streaming requires coordination with frontend
- Cost tracking needs verification with actual token counts

### Mitigations
- Configurable `parallelBatchDelay` to control rate
- `maxParallelism` to cap concurrent requests
- Comprehensive logging for debugging

---

## Testing Strategy

### Unit Tests
1. Dependency graph resolution
2. Circular dependency detection
3. Input mapping resolution
4. Output transformation functions

### Integration Tests
1. Execute simple sequential workflow
2. Execute parallel workflow
3. Execute hybrid workflow (parallel → sequential)
4. Streaming execution with mock Oracle
5. Error handling (step failure, timeout)

### Manual Testing
1. Create CFAS workflow via API
2. Execute with sample interview answer
3. Verify parallel execution via logs
4. Verify streaming via SSE client

---

## Open Questions

1. **Oracle Streaming**: Does `talkwise-oracle` currently support streaming responses? If not, streaming implementation may need to be deferred.

2. **Rate Limits**: What are the actual rate limits for the LLM providers (Claude, GPT-4)? This affects optimal `parallelBatchDelay` values.

3. **Frontend Integration**: How will the Practice Interviews frontend consume streaming workflow results? SSE? WebSocket?

4. **Error Recovery**: Should we support partial workflow recovery (resume from failed step)?

---

## Conclusion

The `talkwise-promptsmith` system already has the foundation for parallel execution via its dependency-graph model. The enhancements proposed here add:

1. **Controlled parallelism** with rate limiting
2. **Streaming support** for real-time feedback
3. **Output transformations** for structured results
4. **Input mapping** for flexible prompt composition

The CFAS workflow demonstrates a ~60% reduction in execution time by running independent prompts in parallel. This pattern extends to any workflow where prompts have independent inputs.

---

*Brief created by Ruk, 2026-02-03*
