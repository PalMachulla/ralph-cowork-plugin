# Architecture Reference

## Option Analysis

### Native Extension Approach

**Concept**: Build as an extension/feature for the Claude Desktop app's Cowork tab.

**Advantages**:
- Leverages existing Cowork sandbox (VZVirtualMachine on macOS)
- Native UI integration
- Anthropic-managed security model
- No separate API costs (uses Max subscription)

**Challenges**:
- Requires Anthropic partnership or internal access
- Plugin system for Cowork doesn't exist yet
- Dependent on Anthropic's roadmap

**Implementation path**:
1. Prototype as proof-of-concept
2. Submit to Anthropic as feature request with working demo
3. Collaborate on native integration

**Technical requirements**:
- macOS development environment
- Swift/Electron knowledge (Claude Desktop is Electron-based)
- Understanding of Cowork's session management

### Wrapper Application Approach

**Concept**: Standalone application that orchestrates Claude API calls with local file access.

**Advantages**:
- Ship independently, no Anthropic dependency
- Full control over UX and features
- Cross-platform potential (Electron/Tauri)
- Can evolve faster than native integration

**Challenges**:
- Requires API key management
- User pays API costs (not subscription-included)
- Must implement own sandbox/security
- Duplicates some Cowork functionality

**Implementation path**:
1. Build desktop app (recommend: Tauri for lightweight, Electron for speed)
2. Implement file system sandbox
3. Create API orchestration layer
4. Add iteration loop logic
5. Ship to users directly

## Recommended: Hybrid Approach

Start with **Wrapper Application** for speed-to-market, design for **Native Extension** compatibility.

```
Phase 1: Wrapper MVP
├── Tauri/Electron desktop app
├── Claude API integration
├── Basic loop mechanism
└── Proof of concept for users

Phase 2: Native Proposal  
├── Document learnings from MVP
├── Create Anthropic feature proposal
├── Demonstrate user demand
└── Offer to contribute code

Phase 3: Native Integration (if accepted)
├── Port logic to Cowork plugin format
├── Collaborate with Anthropic team
└── Deprecate standalone wrapper
```

## Core Components

### 1. Session Manager

Handles Claude API interactions:

```typescript
interface SessionManager {
  // Start a new task session
  startSession(config: TaskConfig): Promise<SessionId>;
  
  // Send iteration to Claude
  runIteration(sessionId: SessionId, context: IterationContext): Promise<IterationResult>;
  
  // Check completion status
  checkCompletion(result: IterationResult, criteria: SuccessCriteria): CompletionStatus;
  
  // Terminate session
  endSession(sessionId: SessionId, reason: EndReason): void;
}

interface TaskConfig {
  prompt: string;
  folderPath: string;
  successCriteria: SuccessCriteria;
  maxIterations: number;
  stuckThreshold: number;
}

interface IterationContext {
  iterationNumber: number;
  previousOutput?: string;
  filesModified: string[];
  progressLog: string;
}

type CompletionStatus = 'continue' | 'complete' | 'stuck' | 'max_iterations';
```

### 2. File System Sandbox

Secure folder access layer:

```typescript
interface FolderSandbox {
  // Set the working folder (user must explicitly grant)
  setWorkingFolder(path: string): Promise<boolean>;
  
  // List files in sandbox
  listFiles(): Promise<FileInfo[]>;
  
  // Read file (only within sandbox)
  readFile(relativePath: string): Promise<Buffer>;
  
  // Write file (only within sandbox)
  writeFile(relativePath: string, content: Buffer): Promise<void>;
  
  // Create backup snapshot
  createSnapshot(): Promise<SnapshotId>;
  
  // Restore from snapshot
  restoreSnapshot(snapshotId: SnapshotId): Promise<void>;
}
```

### 3. Loop Controller

Main orchestration logic:

```typescript
class LoopController {
  async runLoop(config: TaskConfig): AsyncGenerator<LoopEvent> {
    const sandbox = await this.initSandbox(config.folderPath);
    await sandbox.createSnapshot(); // Safety backup
    
    let iteration = 0;
    let stuckCount = 0;
    let lastError: string | null = null;
    
    while (iteration < config.maxIterations) {
      iteration++;
      yield { type: 'iteration_start', iteration };
      
      const context = await this.buildContext(sandbox, iteration);
      const result = await this.session.runIteration(config, context);
      
      yield { type: 'iteration_complete', iteration, result };
      
      const status = this.checkCompletion(result, config.successCriteria);
      
      if (status === 'complete') {
        yield { type: 'loop_complete', iteration, success: true };
        return;
      }
      
      if (status === 'stuck') {
        stuckCount++;
        if (stuckCount >= config.stuckThreshold) {
          yield { type: 'loop_stuck', iteration, reason: result.stuckReason };
          return;
        }
      } else {
        stuckCount = 0; // Reset on progress
      }
      
      await this.updateProgressLog(sandbox, iteration, result);
    }
    
    yield { type: 'max_iterations_reached', iteration };
  }
}
```

### 4. Success Criteria Parser

Detect completion signals in Claude's output:

```typescript
interface CriteriaParser {
  // Extract completion signal
  findCompletionSignal(output: string): string | null;
  
  // Extract stuck signal
  findStuckSignal(output: string): StuckInfo | null;
  
  // Validate against criteria checklist
  validateCriteria(output: string, criteria: SuccessCriteria): ValidationResult;
}

// Signal patterns (configurable)
const DEFAULT_PATTERNS = {
  complete: /<complete>(.+?)<\/complete>/i,
  stuck: /<stuck>(.+?)<\/stuck>/i,
  progress: /<progress>(.+?)<\/progress>/i,
};
```

### 5. Prompt Builder

Construct iteration prompts:

```typescript
function buildIterationPrompt(
  originalTask: string,
  criteria: SuccessCriteria,
  iteration: number,
  maxIterations: number,
  progressLog: string
): string {
  return `
## Task
${originalTask}

## Success Criteria
When ALL criteria are met, output: <complete>TASK_COMPLETE</complete>
${criteria.items.map(c => `- ${c}`).join('\n')}

## If Stuck
If you cannot proceed, output: <stuck>REASON: description</stuck>

## Current Status
Iteration: ${iteration}/${maxIterations}
Progress so far:
${progressLog || 'No previous progress'}

## Instructions
1. Review the current state of files in the working folder
2. Continue working toward the success criteria
3. Update progress.txt with what you accomplished this iteration
4. Either complete the task, make progress, or report if stuck
`.trim();
}
```

## API Integration

### Claude API Configuration

```typescript
const CLAUDE_CONFIG = {
  model: 'claude-sonnet-4-20250514', // or opus for complex tasks
  max_tokens: 4096,
  system: `You are working in an autonomous loop. Each iteration, you:
1. Assess the current state of the working folder
2. Make progress toward the success criteria
3. Signal completion or stuck status using the specified tags
4. Keep progress.txt updated

Be thorough but efficient. Each iteration has API cost.`,
};
```

### Cost Estimation

```typescript
function estimateCost(config: TaskConfig): CostEstimate {
  const avgTokensPerIteration = 3000; // input + output
  const pricePerMillion = 3.00; // Sonnet pricing
  
  const estimatedTokens = avgTokensPerIteration * config.maxIterations;
  const estimatedCost = (estimatedTokens / 1_000_000) * pricePerMillion;
  
  return {
    minCost: estimatedCost * 0.3, // Best case: completes early
    maxCost: estimatedCost,       // Worst case: all iterations
    avgCost: estimatedCost * 0.6, // Typical case
  };
}
```

## Security Considerations

1. **Folder Isolation**: Never access files outside designated folder
2. **No Network by Default**: Claude should not make web requests unless explicitly enabled
3. **Backup Before Modify**: Always snapshot folder state before first iteration
4. **Rate Limiting**: Prevent runaway API calls (max 1 iteration per 10 seconds)
5. **User Confirmation**: Require explicit "Start" action, show cost estimate first
6. **Audit Log**: Record all file operations for user review
