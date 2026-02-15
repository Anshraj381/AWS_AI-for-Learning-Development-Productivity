# Design Document: DevSim Dashboard

## Overview

The DevSim Dashboard is an interactive AI-powered learning platform that simulates a realistic software development environment. Users practice fixing broken code while receiving feedback from an AI Senior Developer Agent named "Sarah Chen." The system combines a glassmorphism-styled Kanban dashboard with an integrated Monaco code editor, creating an immersive learning experience.

### Core Design Principles

1. **Clean Architecture**: Clear separation between presentation, business logic, and data layers
2. **Modular Components**: Single-responsibility components that are reusable and testable
3. **Type Safety**: Full TypeScript implementation with strict type checking
4. **Scalable State Management**: Centralized state using React Context API
5. **Responsible AI**: AI provides guidance without solving problems completely

### Technology Stack

- **Frontend Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS with glassmorphism utilities
- **Code Editor**: Monaco Editor (@monaco-editor/react)
- **AI Integration**: AWS Bedrock or OpenAI API
- **State Management**: React Context API
- **Animations**: Framer Motion
- **Icons**: Lucide React
- **Charts**: Recharts for analytics visualization
- **Celebrations**: react-confetti for success animations

## Architecture

### High-Level Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        Dashboard[Dashboard Page]
        Editor[Editor Page]
        Analytics[Analytics View]
    end
    
    subgraph "Component Layer"
        KanbanBoard[Kanban Board]
        TaskCard[Task Card]
        CodeEditor[Monaco Editor]
        ReviewModal[Review Modal]
        ProgressChart[Progress Chart]
        AnalyticsGraph[Analytics Graph]
    end
    
    subgraph "Business Logic Layer"
        TaskManager[Task Manager]
        ReviewService[Review Service]
        ProgressTracker[Progress Tracker]
        AnalyticsEngine[Analytics Engine]
    end
    
    subgraph "API Layer"
        ReviewAPI[/api/review]
    end
    
    subgraph "External Services"
        LLM[AWS Bedrock / OpenAI]
    end
    
    subgraph "Data Layer"
        LocalStorage[Browser LocalStorage]
        ChallengeData[Challenge Data]
    end
    
    Dashboard --> KanbanBoard
    Dashboard --> ProgressChart
    Dashboard --> Analytics
    Editor --> CodeEditor
    Editor --> ReviewModal
    
    KanbanBoard --> TaskCard
    TaskCard --> TaskManager
    CodeEditor --> ReviewService
    
    ReviewService --> ReviewAPI
    ReviewAPI --> LLM
    
    TaskManager --> LocalStorage
    ProgressTracker --> LocalStorage
    AnalyticsEngine --> LocalStorage
    
    TaskManager --> ChallengeData
    ReviewService --> ChallengeData
```

### Directory Structure

```
app/
├── dashboard/
│   └── page.tsx                 # Main dashboard with Kanban board
├── editor/
│   └── [taskId]/
│       └── page.tsx             # Code editor workstation
├── api/
│   └── review/
│       └── route.ts             # AI code review endpoint
└── layout.tsx                   # Root layout with providers

components/
├── dashboard/
│   ├── KanbanBoard.tsx          # Main board component
│   ├── TaskCard.tsx             # Individual task cards
│   ├── ProgressCircle.tsx       # Circular progress chart
│   └── Sidebar.tsx              # Navigation sidebar
├── editor/
│   ├── TicketPanel.tsx          # Left panel with requirements
│   ├── CodeEditor.tsx           # Monaco editor wrapper
│   ├── SubmitButton.tsx         # Floating submit button
│   └── ReviewModal.tsx          # Code review feedback modal
├── analytics/
│   ├── ScoreGraph.tsx           # Score progression chart
│   ├── RubricBreakdown.tsx      # Category score visualization
│   └── ImprovementDelta.tsx     # Attempt comparison
└── shared/
    ├── GlassCard.tsx            # Reusable glassmorphism card
    └── ConfettiCelebration.tsx  # Success animation

lib/
├── challenges.ts                # Hardcoded challenge data
├── ai/
│   ├── reviewer.ts              # AI agent logic
│   ├── prompts.ts               # System prompts
│   └── rubric.ts                # Scoring rubric logic
├── state/
│   ├── TaskContext.tsx          # Task state management
│   ├── ProgressContext.tsx      # Progress tracking state
│   └── AnalyticsContext.tsx     # Analytics state
└── utils/
    ├── storage.ts               # LocalStorage utilities
    └── validation.ts            # Input validation

types/
├── task.ts                      # Task and challenge types
├── review.ts                    # Review response types
├── analytics.ts                 # Analytics data types
└── rubric.ts                    # Rubric scoring types
```



## Components and Interfaces

### Dashboard Components

#### KanbanBoard Component

**Purpose**: Displays tasks organized in four columns representing workflow states.

**Props**:
```typescript
interface KanbanBoardProps {
  tasks: Task[];
  onTaskClick: (taskId: string) => void;
}
```

**Behavior**:
- Renders four columns: Backlog, Current Sprint, In Review, Production
- Filters tasks by status and displays in appropriate column
- Applies glassmorphism styling to column containers
- Handles task card click events for navigation

**Styling**:
- Background: `bg-slate-950`
- Column containers: `bg-white/5 backdrop-blur-xl`
- Grid layout with responsive columns

#### TaskCard Component

**Purpose**: Displays individual task information with visual indicators.

**Props**:
```typescript
interface TaskCardProps {
  task: Task;
  onClick?: () => void;
  locked?: boolean;
}
```

**Behavior**:
- Displays ticket ID, difficulty badge, and XP reward
- Shows locked state for backlog tasks
- Applies hover effects for interactive cards
- Uses Framer Motion for smooth animations

**Visual States**:
- **Locked**: Opacity reduced, cursor not-allowed
- **Available**: Full opacity, cursor pointer, hover scale effect
- **In Review**: Yellow border accent
- **Completed**: Green checkmark icon

#### ProgressCircle Component

**Purpose**: Visualizes internship completion percentage.

**Props**:
```typescript
interface ProgressCircleProps {
  percentage: number;
  totalTasks: number;
  completedTasks: number;
}
```

**Implementation**:
- Uses Recharts RadialBarChart
- Displays percentage in center
- Color gradient from purple to green based on progress
- Animated transitions when percentage updates

#### Sidebar Component

**Purpose**: Provides navigation and user context.

**Features**:
- Logo and branding
- Navigation links (Dashboard, Analytics)
- User profile section
- Glassmorphism styling consistent with theme

### Editor Components

#### TicketPanel Component

**Purpose**: Displays task requirements and context in the left panel.

**Props**:
```typescript
interface TicketPanelProps {
  task: Task;
}
```

**Layout**:
- Fixed 30% viewport width
- Scrollable content area
- Sections: Context, Requirements, API Documentation
- Glassmorphism card styling

**Content Structure**:
```typescript
interface TicketContent {
  ticketId: string;
  title: string;
  context: string;
  requirements: string[];
  apiDocs?: string;
  difficulty: 'Junior' | 'Mid' | 'Senior';
  xpReward: number;
}
```

#### CodeEditor Component

**Purpose**: Wraps Monaco Editor with custom configuration.

**Props**:
```typescript
interface CodeEditorProps {
  initialCode: string;
  language: string;
  onChange: (value: string) => void;
  readOnly?: boolean;
}
```

**Configuration**:
```typescript
const editorOptions = {
  fontSize: 14,
  fontFamily: 'JetBrains Mono',
  theme: 'vs-dark',
  minimap: { enabled: true },
  scrollBeyondLastLine: false,
  automaticLayout: true,
  tabSize: 2,
  wordWrap: 'on'
};
```

**Behavior**:
- Loads with pre-populated buggy code
- Provides syntax highlighting based on language
- Captures code changes via onChange callback
- Supports full IDE features (autocomplete, error detection)

#### SubmitButton Component

**Purpose**: Floating action button to submit code for review.

**Props**:
```typescript
interface SubmitButtonProps {
  onSubmit: () => void;
  loading?: boolean;
}
```

**Styling**:
- Fixed position: bottom-right of editor panel
- Gradient purple background: `bg-gradient-to-r from-purple-500 to-pink-500`
- Hover effect with scale animation
- Loading spinner when processing

#### ReviewModal Component

**Purpose**: Displays AI code review feedback in a modal overlay.

**Props**:
```typescript
interface ReviewModalProps {
  isOpen: boolean;
  onClose: () => void;
  review: ReviewResponse;
}
```

**Layout Sections**:
1. **Header**: Status badge (APPROVED/REJECTED) and total score
2. **Rubric Breakdown**: Six category scores with visual bars
3. **Feedback Summary**: Overall feedback text
4. **Line Comments**: Specific code issues with line numbers
5. **Diff Suggestions**: Recommended code improvements
6. **Action Buttons**: Close or Retry

**Styling**:
- Modal overlay: `bg-black/50 backdrop-blur-sm`
- Content card: Glassmorphism with `bg-slate-900/95`
- Status colors: Green for approved, red for rejected

### Analytics Components

#### ScoreGraph Component

**Purpose**: Visualizes score progression over time.

**Props**:
```typescript
interface ScoreGraphProps {
  attempts: AttemptHistory[];
}
```

**Implementation**:
- Uses Recharts LineChart
- X-axis: Attempt number or timestamp
- Y-axis: Total score (0-60)
- Shows trend line with data points
- Highlights improvement areas

#### RubricBreakdown Component

**Purpose**: Displays category scores with visual indicators.

**Props**:
```typescript
interface RubricBreakdownProps {
  rubric: RubricScores;
  highlightStrongest?: boolean;
  highlightWeakest?: boolean;
}
```

**Visualization**:
- Horizontal bar chart for each category
- Color coding: Green (8-10), Yellow (6-7), Red (0-5)
- Highlights strongest and weakest categories
- Shows score out of 10 for each category

#### ImprovementDelta Component

**Purpose**: Shows improvement between attempts.

**Props**:
```typescript
interface ImprovementDeltaProps {
  previousScore: number;
  currentScore: number;
}
```

**Display**:
- Delta value with +/- indicator
- Color: Green for improvement, red for decline
- Percentage improvement calculation
- Animated number transitions



## Data Models

### Task Model

```typescript
interface Task {
  id: string;
  ticketId: string;
  title: string;
  difficulty: 'Junior' | 'Mid' | 'Senior';
  xpReward: number;
  status: 'backlog' | 'current_sprint' | 'in_review' | 'production';
  context: string;
  requirements: string[];
  apiDocs?: string;
  buggyCode: string;
  language: 'typescript' | 'javascript' | 'python';
  expectedSolution?: string;
  type: 'standard' | 'production_incident';
}
```

### Challenge Data Structure

```typescript
interface Challenge {
  level: number;
  task: Task;
  hints: string[];
  commonMistakes: string[];
}

// Example challenge data
const challenges: Challenge[] = [
  {
    level: 1,
    task: {
      id: 'task-001',
      ticketId: 'DEV-101',
      title: 'Fix the React Counter',
      difficulty: 'Junior',
      xpReward: 100,
      status: 'current_sprint',
      context: 'The counter component is not incrementing when the button is clicked.',
      requirements: [
        'Add onClick handler to the button',
        'Ensure state updates correctly',
        'Verify counter displays updated value'
      ],
      buggyCode: `
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button>Increment</button>
    </div>
  );
}
      `,
      language: 'typescript',
      type: 'standard'
    },
    hints: [
      'Check if the button has an event handler',
      'Look at how setCount should be called'
    ],
    commonMistakes: [
      'Forgetting to add onClick handler',
      'Calling setCount without a function'
    ]
  },
  {
    level: 2,
    task: {
      id: 'task-002',
      ticketId: 'DEV-202',
      title: 'Secure the API Endpoint',
      difficulty: 'Mid',
      xpReward: 250,
      status: 'backlog',
      context: 'The API endpoint is exposing sensitive credentials in the code.',
      requirements: [
        'Move API key to environment variable',
        'Use process.env to access the key',
        'Add error handling for missing env vars'
      ],
      apiDocs: 'Use process.env.API_KEY to access the key',
      buggyCode: `
export async function GET(request: Request) {
  const apiKey = 'sk-1234567890abcdef';
  
  const response = await fetch('https://api.example.com/data', {
    headers: {
      'Authorization': \`Bearer \${apiKey}\`
    }
  });
  
  return Response.json(await response.json());
}
      `,
      language: 'typescript',
      type: 'standard'
    },
    hints: [
      'Never hardcode API keys',
      'Use environment variables for sensitive data'
    ],
    commonMistakes: [
      'Leaving hardcoded credentials',
      'Not validating environment variables exist'
    ]
  },
  {
    level: 3,
    task: {
      id: 'task-003',
      ticketId: 'DEV-303',
      title: 'Optimize Search Algorithm',
      difficulty: 'Senior',
      xpReward: 500,
      status: 'backlog',
      context: 'The search function is causing performance issues with large datasets.',
      requirements: [
        'Optimize from O(n²) to O(n) complexity',
        'Use a Set or Map for efficient lookups',
        'Maintain the same functionality'
      ],
      buggyCode: `
function findDuplicates(arr: number[]): number[] {
  const duplicates: number[] = [];
  
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j] && !duplicates.includes(arr[i])) {
        duplicates.push(arr[i]);
      }
    }
  }
  
  return duplicates;
}
      `,
      language: 'typescript',
      type: 'production_incident'
    },
    hints: [
      'Consider using a Set to track seen values',
      'Single pass through the array is possible'
    ],
    commonMistakes: [
      'Still using nested loops',
      'Not handling edge cases'
    ]
  }
];
```

### Review Response Model

```typescript
interface ReviewResponse {
  total_score: number;
  rubric: RubricScores;
  status: 'APPROVED' | 'REJECTED';
  feedback_summary: string;
  line_comments: LineComment[];
  diff_suggestions: DiffSuggestion[];
}

interface RubricScores {
  functional_correctness: number;
  code_readability: number;
  structure_modularity: number;
  performance_efficiency: number;
  security_practices: number;
  error_handling: number;
}

interface LineComment {
  line_number: number;
  comment: string;
  severity: 'error' | 'warning' | 'info';
}

interface DiffSuggestion {
  original: string;
  suggested: string;
  explanation: string;
}
```

### Progress Tracking Model

```typescript
interface UserProgress {
  completedTasks: string[];
  totalXP: number;
  currentLevel: number;
  taskAttempts: Map<string, AttemptHistory[]>;
}

interface AttemptHistory {
  taskId: string;
  timestamp: Date;
  totalScore: number;
  rubricScores: RubricScores;
  status: 'APPROVED' | 'REJECTED';
  codeSubmitted: string;
}
```

### Analytics Model

```typescript
interface AnalyticsData {
  scoreProgression: ScoreDataPoint[];
  categoryAverages: RubricScores;
  strongestCategory: keyof RubricScores;
  weakestCategory: keyof RubricScores;
  improvementRate: number;
  totalAttempts: number;
  successRate: number;
}

interface ScoreDataPoint {
  attemptNumber: number;
  timestamp: Date;
  totalScore: number;
  taskId: string;
}
```



## AI Reviewer Implementation

### AI Agent Persona

**Name**: Sarah Chen, Senior Staff Engineer

**Personality Traits**:
- Professional and constructive
- Brief and to-the-point
- Focuses on teaching, not just correcting
- Emphasizes best practices and clean code
- Maintains high standards for code quality

### System Prompt Design

```typescript
const REVIEWER_SYSTEM_PROMPT = `
You are Sarah Chen, a Senior Staff Engineer conducting a code review.

Your role is to evaluate code submissions against the provided requirements and provide constructive feedback.

Evaluation Criteria:
1. Functional Correctness (0-10): Does the code solve the problem?
2. Code Readability (0-10): Is the code easy to understand?
3. Structure and Modularity (0-10): Is the code well-organized?
4. Performance Efficiency (0-10): Is the code optimized?
5. Security Best Practices (0-10): Are there security vulnerabilities?
6. Error Handling (0-10): Does the code handle errors properly?

Approval Thresholds:
- Functional Correctness must be >= 7
- Security Best Practices must be >= 6
- Total Score must be >= 30

Guidelines:
- Reject code with poor variable naming, missing comments, or messy structure even if functionally correct
- Provide specific line-by-line feedback
- Explain WHY code is wrong (security, performance, clean code issues)
- Offer hints and guidance, NOT complete solutions
- Be professional, brief, and constructive
- For production incidents, require Performance Efficiency >= 8

Response Format:
Return a JSON object with:
- total_score: sum of all category scores (0-60)
- rubric: object with six category scores
- status: "APPROVED" or "REJECTED"
- feedback_summary: overall feedback (2-3 sentences)
- line_comments: array of specific issues with line numbers
- diff_suggestions: array of improvement suggestions
`;
```

### Rubric Scoring Logic

```typescript
interface RubricEvaluator {
  evaluateFunctionalCorrectness(code: string, requirements: string[]): number;
  evaluateReadability(code: string): number;
  evaluateStructure(code: string): number;
  evaluatePerformance(code: string, taskType: string): number;
  evaluateSecurity(code: string): number;
  evaluateErrorHandling(code: string): number;
}

class AIReviewerService implements RubricEvaluator {
  private llmClient: LLMClient;
  
  async reviewCode(
    userCode: string,
    requirements: string[],
    taskType: 'standard' | 'production_incident'
  ): Promise<ReviewResponse> {
    const prompt = this.buildReviewPrompt(userCode, requirements, taskType);
    const response = await this.llmClient.complete(prompt);
    const review = this.parseReviewResponse(response);
    
    // Apply approval logic
    review.status = this.determineStatus(review.rubric, taskType);
    
    return review;
  }
  
  private determineStatus(
    rubric: RubricScores,
    taskType: string
  ): 'APPROVED' | 'REJECTED' {
    const totalScore = Object.values(rubric).reduce((sum, score) => sum + score, 0);
    
    // Check thresholds
    if (rubric.functional_correctness < 7) return 'REJECTED';
    if (rubric.security_practices < 6) return 'REJECTED';
    if (totalScore < 30) return 'REJECTED';
    
    // Production incidents require high performance score
    if (taskType === 'production_incident' && rubric.performance_efficiency < 8) {
      return 'REJECTED';
    }
    
    return 'APPROVED';
  }
  
  private buildReviewPrompt(
    code: string,
    requirements: string[],
    taskType: string
  ): string {
    return `
${REVIEWER_SYSTEM_PROMPT}

Task Type: ${taskType}

Requirements:
${requirements.map((req, i) => `${i + 1}. ${req}`).join('\n')}

User Code:
\`\`\`
${code}
\`\`\`

Provide your review as a JSON object.
    `;
  }
  
  private parseReviewResponse(response: string): ReviewResponse {
    try {
      const parsed = JSON.parse(response);
      
      // Validate structure
      if (!this.isValidReviewResponse(parsed)) {
        throw new Error('Invalid review response structure');
      }
      
      return parsed;
    } catch (error) {
      // Return error response
      return this.createErrorResponse();
    }
  }
  
  private isValidReviewResponse(obj: any): boolean {
    return (
      typeof obj.total_score === 'number' &&
      obj.rubric &&
      typeof obj.status === 'string' &&
      typeof obj.feedback_summary === 'string' &&
      Array.isArray(obj.line_comments) &&
      Array.isArray(obj.diff_suggestions)
    );
  }
  
  private createErrorResponse(): ReviewResponse {
    return {
      total_score: 0,
      rubric: {
        functional_correctness: 0,
        code_readability: 0,
        structure_modularity: 0,
        performance_efficiency: 0,
        security_practices: 0,
        error_handling: 0
      },
      status: 'REJECTED',
      feedback_summary: 'An error occurred while processing your review. Please try again.',
      line_comments: [],
      diff_suggestions: []
    };
  }
}
```

### LLM Integration

```typescript
// AWS Bedrock Integration
class BedrockLLMClient implements LLMClient {
  private client: BedrockRuntimeClient;
  
  constructor() {
    this.client = new BedrockRuntimeClient({
      region: process.env.AWS_REGION || 'us-east-1'
    });
  }
  
  async complete(prompt: string): Promise<string> {
    const command = new InvokeModelCommand({
      modelId: 'anthropic.claude-3-sonnet-20240229-v1:0',
      contentType: 'application/json',
      accept: 'application/json',
      body: JSON.stringify({
        anthropic_version: 'bedrock-2023-05-31',
        max_tokens: 2000,
        messages: [
          {
            role: 'user',
            content: prompt
          }
        ]
      })
    });
    
    const response = await this.client.send(command);
    const responseBody = JSON.parse(new TextDecoder().decode(response.body));
    
    return responseBody.content[0].text;
  }
}

// OpenAI Integration (Alternative)
class OpenAILLMClient implements LLMClient {
  private client: OpenAI;
  
  constructor() {
    this.client = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async complete(prompt: string): Promise<string> {
    const completion = await this.client.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: REVIEWER_SYSTEM_PROMPT
        },
        {
          role: 'user',
          content: prompt
        }
      ],
      response_format: { type: 'json_object' },
      temperature: 0.7,
      max_tokens: 2000
    });
    
    return completion.choices[0].message.content || '';
  }
}
```

## API Routes

### POST /api/review

**Purpose**: Processes code submissions and returns AI review feedback.

**Request Body**:
```typescript
interface ReviewRequest {
  userCode: string;
  ticketRequirements: string[];
  taskId: string;
  taskType: 'standard' | 'production_incident';
}
```

**Response**:
```typescript
interface ReviewAPIResponse {
  success: boolean;
  data?: ReviewResponse;
  error?: string;
}
```

**Implementation**:
```typescript
// app/api/review/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { AIReviewerService } from '@/lib/ai/reviewer';

export async function POST(request: NextRequest) {
  try {
    // Parse request body
    const body = await request.json();
    const { userCode, ticketRequirements, taskId, taskType } = body;
    
    // Validate inputs
    if (!userCode || !ticketRequirements || !taskId) {
      return NextResponse.json(
        { success: false, error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    // Initialize reviewer service
    const reviewer = new AIReviewerService();
    
    // Process review
    const review = await reviewer.reviewCode(
      userCode,
      ticketRequirements,
      taskType || 'standard'
    );
    
    // Return response
    return NextResponse.json({
      success: true,
      data: review
    });
    
  } catch (error) {
    console.error('Review API error:', error);
    
    return NextResponse.json(
      { 
        success: false, 
        error: 'An error occurred while processing your review' 
      },
      { status: 500 }
    );
  }
}
```

**Error Handling**:
- 400: Missing or invalid request parameters
- 500: Internal server error or LLM API failure
- Timeout: 10 second timeout for LLM responses

**Rate Limiting** (Future Enhancement):
- Implement rate limiting to prevent abuse
- Consider caching for identical code submissions



## State Management

### Task Context

```typescript
interface TaskContextValue {
  tasks: Task[];
  currentTask: Task | null;
  setCurrentTask: (taskId: string) => void;
  updateTaskStatus: (taskId: string, status: Task['status']) => void;
  loadTasks: () => void;
}

const TaskContext = createContext<TaskContextValue | undefined>(undefined);

export function TaskProvider({ children }: { children: React.ReactNode }) {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [currentTask, setCurrentTaskState] = useState<Task | null>(null);
  
  useEffect(() => {
    loadTasks();
  }, []);
  
  const loadTasks = () => {
    // Load from challenges data
    const challengeTasks = challenges.map(c => c.task);
    
    // Merge with saved progress from localStorage
    const savedProgress = loadProgressFromStorage();
    const tasksWithProgress = challengeTasks.map(task => ({
      ...task,
      status: savedProgress[task.id]?.status || task.status
    }));
    
    setTasks(tasksWithProgress);
  };
  
  const setCurrentTask = (taskId: string) => {
    const task = tasks.find(t => t.id === taskId);
    setCurrentTaskState(task || null);
  };
  
  const updateTaskStatus = (taskId: string, status: Task['status']) => {
    setTasks(prev => prev.map(task =>
      task.id === taskId ? { ...task, status } : task
    ));
    
    // Persist to localStorage
    saveTaskStatusToStorage(taskId, status);
  };
  
  return (
    <TaskContext.Provider value={{
      tasks,
      currentTask,
      setCurrentTask,
      updateTaskStatus,
      loadTasks
    }}>
      {children}
    </TaskContext.Provider>
  );
}

export function useTaskContext() {
  const context = useContext(TaskContext);
  if (!context) {
    throw new Error('useTaskContext must be used within TaskProvider');
  }
  return context;
}
```

### Progress Context

```typescript
interface ProgressContextValue {
  totalXP: number;
  completedTasks: string[];
  completionPercentage: number;
  addXP: (amount: number) => void;
  markTaskComplete: (taskId: string) => void;
}

const ProgressContext = createContext<ProgressContextValue | undefined>(undefined);

export function ProgressProvider({ children }: { children: React.ReactNode }) {
  const [totalXP, setTotalXP] = useState(0);
  const [completedTasks, setCompletedTasks] = useState<string[]>([]);
  
  useEffect(() => {
    // Load from localStorage
    const saved = loadProgressFromStorage();
    setTotalXP(saved.totalXP || 0);
    setCompletedTasks(saved.completedTasks || []);
  }, []);
  
  const addXP = (amount: number) => {
    setTotalXP(prev => {
      const newTotal = prev + amount;
      saveProgressToStorage({ totalXP: newTotal });
      return newTotal;
    });
  };
  
  const markTaskComplete = (taskId: string) => {
    setCompletedTasks(prev => {
      if (prev.includes(taskId)) return prev;
      const updated = [...prev, taskId];
      saveProgressToStorage({ completedTasks: updated });
      return updated;
    });
  };
  
  const completionPercentage = useMemo(() => {
    const totalTasks = challenges.length;
    return Math.round((completedTasks.length / totalTasks) * 100);
  }, [completedTasks]);
  
  return (
    <ProgressContext.Provider value={{
      totalXP,
      completedTasks,
      completionPercentage,
      addXP,
      markTaskComplete
    }}>
      {children}
    </ProgressContext.Provider>
  );
}
```

### Analytics Context

```typescript
interface AnalyticsContextValue {
  analyticsData: AnalyticsData;
  recordAttempt: (attempt: AttemptHistory) => void;
  getTaskHistory: (taskId: string) => AttemptHistory[];
  getImprovementDelta: (taskId: string) => number | null;
}

const AnalyticsContext = createContext<AnalyticsContextValue | undefined>(undefined);

export function AnalyticsProvider({ children }: { children: React.ReactNode }) {
  const [attempts, setAttempts] = useState<AttemptHistory[]>([]);
  
  useEffect(() => {
    const saved = loadAnalyticsFromStorage();
    setAttempts(saved.attempts || []);
  }, []);
  
  const recordAttempt = (attempt: AttemptHistory) => {
    setAttempts(prev => {
      const updated = [...prev, attempt];
      saveAnalyticsToStorage({ attempts: updated });
      return updated;
    });
  };
  
  const getTaskHistory = (taskId: string) => {
    return attempts.filter(a => a.taskId === taskId);
  };
  
  const getImprovementDelta = (taskId: string) => {
    const history = getTaskHistory(taskId);
    if (history.length < 2) return null;
    
    const latest = history[history.length - 1];
    const previous = history[history.length - 2];
    
    return latest.totalScore - previous.totalScore;
  };
  
  const analyticsData = useMemo(() => {
    return calculateAnalytics(attempts);
  }, [attempts]);
  
  return (
    <AnalyticsContext.Provider value={{
      analyticsData,
      recordAttempt,
      getTaskHistory,
      getImprovementDelta
    }}>
      {children}
    </AnalyticsContext.Provider>
  );
}

function calculateAnalytics(attempts: AttemptHistory[]): AnalyticsData {
  if (attempts.length === 0) {
    return {
      scoreProgression: [],
      categoryAverages: {
        functional_correctness: 0,
        code_readability: 0,
        structure_modularity: 0,
        performance_efficiency: 0,
        security_practices: 0,
        error_handling: 0
      },
      strongestCategory: 'functional_correctness',
      weakestCategory: 'functional_correctness',
      improvementRate: 0,
      totalAttempts: 0,
      successRate: 0
    };
  }
  
  const scoreProgression = attempts.map((attempt, index) => ({
    attemptNumber: index + 1,
    timestamp: attempt.timestamp,
    totalScore: attempt.totalScore,
    taskId: attempt.taskId
  }));
  
  const categoryAverages = calculateCategoryAverages(attempts);
  const strongestCategory = findStrongestCategory(categoryAverages);
  const weakestCategory = findWeakestCategory(categoryAverages);
  
  const approvedAttempts = attempts.filter(a => a.status === 'APPROVED').length;
  const successRate = (approvedAttempts / attempts.length) * 100;
  
  const improvementRate = calculateImprovementRate(attempts);
  
  return {
    scoreProgression,
    categoryAverages,
    strongestCategory,
    weakestCategory,
    improvementRate,
    totalAttempts: attempts.length,
    successRate
  };
}
```

## LocalStorage Utilities

```typescript
// lib/utils/storage.ts

const STORAGE_KEYS = {
  PROGRESS: 'devsim_progress',
  ANALYTICS: 'devsim_analytics',
  TASK_STATUS: 'devsim_task_status'
};

export function loadProgressFromStorage(): UserProgress {
  if (typeof window === 'undefined') return getDefaultProgress();
  
  try {
    const saved = localStorage.getItem(STORAGE_KEYS.PROGRESS);
    return saved ? JSON.parse(saved) : getDefaultProgress();
  } catch (error) {
    console.error('Error loading progress:', error);
    return getDefaultProgress();
  }
}

export function saveProgressToStorage(progress: Partial<UserProgress>): void {
  if (typeof window === 'undefined') return;
  
  try {
    const current = loadProgressFromStorage();
    const updated = { ...current, ...progress };
    localStorage.setItem(STORAGE_KEYS.PROGRESS, JSON.stringify(updated));
  } catch (error) {
    console.error('Error saving progress:', error);
  }
}

export function loadAnalyticsFromStorage(): { attempts: AttemptHistory[] } {
  if (typeof window === 'undefined') return { attempts: [] };
  
  try {
    const saved = localStorage.getItem(STORAGE_KEYS.ANALYTICS);
    return saved ? JSON.parse(saved) : { attempts: [] };
  } catch (error) {
    console.error('Error loading analytics:', error);
    return { attempts: [] };
  }
}

export function saveAnalyticsToStorage(data: { attempts: AttemptHistory[] }): void {
  if (typeof window === 'undefined') return;
  
  try {
    localStorage.setItem(STORAGE_KEYS.ANALYTICS, JSON.stringify(data));
  } catch (error) {
    console.error('Error saving analytics:', error);
  }
}

export function saveTaskStatusToStorage(taskId: string, status: Task['status']): void {
  if (typeof window === 'undefined') return;
  
  try {
    const saved = localStorage.getItem(STORAGE_KEYS.TASK_STATUS);
    const taskStatuses = saved ? JSON.parse(saved) : {};
    taskStatuses[taskId] = { status, updatedAt: new Date().toISOString() };
    localStorage.setItem(STORAGE_KEYS.TASK_STATUS, JSON.stringify(taskStatuses));
  } catch (error) {
    console.error('Error saving task status:', error);
  }
}

function getDefaultProgress(): UserProgress {
  return {
    completedTasks: [],
    totalXP: 0,
    currentLevel: 1,
    taskAttempts: new Map()
  };
}
```



## Correctness Properties

A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.

### UI Component Properties

Property 1: Kanban board column structure
*For any* Kanban board render, the board should display exactly four columns with labels "Backlog", "Current Sprint", "In Review", and "Production"
**Validates: Requirements 1.2**

Property 2: Task card information completeness
*For any* task rendered as a card, the card should display the ticket ID, difficulty level, and XP reward amount
**Validates: Requirements 1.3, 2.1, 2.2, 2.3**

Property 3: Glassmorphism styling consistency
*For any* card element in the UI, it should have the glassmorphism styling classes (bg-white/5 and backdrop-blur-xl)
**Validates: Requirements 1.5, 9.2**

Property 4: Backlog task interaction prevention
*For any* task with status "backlog", the task card should prevent user interaction (disabled state or no click handler)
**Validates: Requirements 2.4, 10.5**

Property 5: Current sprint task navigation
*For any* task with status "current_sprint", clicking the task card should trigger navigation to the editor page with that task's ID
**Validates: Requirements 2.5, 10.1**

Property 6: Ticket panel content completeness
*For any* task displayed in the ticket panel, the panel should show the context, requirements, and API documentation (if present)
**Validates: Requirements 3.4**

Property 7: Editor code initialization
*For any* task loaded in the editor, the Monaco editor should be pre-loaded with that task's buggy code
**Validates: Requirements 3.5**

Property 8: Submit button code capture
*For any* editor state, clicking the Submit PR button should capture the exact current content of the editor
**Validates: Requirements 4.5**

### API and Review Properties

Property 9: Review API request structure
*For any* code submission, the request to /api/review should include userCode, ticketRequirements, and taskId fields
**Validates: Requirements 5.1**

Property 10: Review response structure completeness
*For any* review response, it should contain all required fields: total_score, rubric, status, feedback_summary, line_comments, and diff_suggestions
**Validates: Requirements 5.3, 11.4, 11A.3, 11A.4, 11A.5, 11A.6**

Property 11: Review response timeout
*For any* code submission, the review API should respond within 10 seconds
**Validates: Requirements 5.4**

Property 12: Rejected review modal display
*For any* review response with status "REJECTED", the system should display the code review modal with feedback
**Validates: Requirements 5.5**

Property 13: Approved review celebration
*For any* review response with status "APPROVED", the system should display a confetti animation and redirect to the dashboard
**Validates: Requirements 5.6, 12.3**

Property 14: API request validation
*For any* API request missing userCode or ticketRequirements, the endpoint should return a 400 error response
**Validates: Requirements 11.2**

Property 15: API error handling
*For any* API request that fails during processing, the endpoint should return an error response with appropriate HTTP status code
**Validates: Requirements 11.5**

Property 16: Malformed response handling
*For any* malformed review response, the system should handle the error gracefully and display a user-friendly error message
**Validates: Requirements 11A.7**

### Rubric and Scoring Properties

Property 17: Rubric category completeness
*For any* review response, the rubric object should contain all six scoring categories: functional_correctness, code_readability, structure_modularity, performance_efficiency, security_practices, and error_handling
**Validates: Requirements 6A.1, 11A.2**

Property 18: Rubric score bounds
*For any* review response, all rubric category scores should be numbers between 0 and 10 inclusive
**Validates: Requirements 6A.2**

Property 19: Total score calculation
*For any* review response, the total_score should equal the sum of all six rubric category scores
**Validates: Requirements 6A.3**

Property 20: Total score bounds
*For any* review response, the total_score should be a number between 0 and 60 inclusive
**Validates: Requirements 11A.1**

Property 21: Functional correctness rejection threshold
*For any* review with functional_correctness score less than 7, the status should be "REJECTED"
**Validates: Requirements 6A.4**

Property 22: Security rejection threshold
*For any* review with security_practices score less than 6, the status should be "REJECTED"
**Validates: Requirements 6A.5**

Property 23: Total score rejection threshold
*For any* review with total_score less than 30, the status should be "REJECTED"
**Validates: Requirements 6A.6**

Property 24: Approval threshold satisfaction
*For any* review where functional_correctness >= 7, security_practices >= 6, and total_score >= 30, the status should be "APPROVED"
**Validates: Requirements 6A.7**

Property 25: Production incident performance threshold
*For any* production_incident task review with performance_efficiency score less than 8, the status should be "REJECTED"
**Validates: Requirements 13.5**

Property 26: Rubric display in modal
*For any* review modal displayed, the rubric scores should be visible in the rendered output
**Validates: Requirements 6A.8**

### Challenge Data Properties

Property 27: Challenge data structure completeness
*For any* challenge loaded, it should provide buggy code, context, requirements, and expected solution criteria
**Validates: Requirements 7.5**

Property 28: Production incident context
*For any* production_incident challenge, the context should describe production symptoms (performance issues, errors, etc.)
**Validates: Requirements 13.2, 13.3**

### Progress and State Management Properties

Property 29: XP award on completion
*For any* task completed with APPROVED status, the user's total XP should increase by exactly the task's xpReward amount
**Validates: Requirements 8.1**

Property 30: Completion percentage calculation
*For any* set of tasks, the completion percentage should equal (completedTasks.length / totalTasks) * 100
**Validates: Requirements 8.2**

Property 31: Task status transition on approval
*For any* task with an approved review, the task status should transition from "in_review" to "production"
**Validates: Requirements 8.3, 10.3**

Property 32: Task status transition on submission
*For any* task when PR is submitted, the task status should transition to "in_review"
**Validates: Requirements 10.2**

Property 33: Task status transition on rejection
*For any* task with a rejected review, the task status should transition back to "current_sprint"
**Validates: Requirements 10.4**

Property 34: Progress persistence round-trip
*For any* progress data (XP, completed tasks), saving to localStorage then loading should produce equivalent data
**Validates: Requirements 8.5**

### Analytics Properties

Property 35: Attempt history tracking
*For any* task attempt, the attempt should be recorded in history with timestamp, totalScore, rubricScores, and status
**Validates: Requirements 16.1**

Property 36: Improvement delta calculation
*For any* task with at least two attempts, the improvement delta should equal (latest.totalScore - previous.totalScore)
**Validates: Requirements 16.3**

Property 37: Strongest category identification
*For any* set of attempts, the strongest category should be the rubric category with the highest average score
**Validates: Requirements 16.4**

Property 38: Weakest category identification
*For any* set of attempts, the weakest category should be the rubric category with the lowest average score
**Validates: Requirements 16.5**

Property 39: Analytics persistence round-trip
*For any* analytics data (attempts, scores), saving to localStorage then loading should produce equivalent data
**Validates: Requirements 16.6**



## Error Handling

### Client-Side Error Handling

#### API Request Failures

```typescript
async function submitCodeForReview(
  code: string,
  requirements: string[],
  taskId: string,
  taskType: string
): Promise<ReviewResponse> {
  try {
    const response = await fetch('/api/review', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        userCode: code,
        ticketRequirements: requirements,
        taskId,
        taskType
      }),
      signal: AbortSignal.timeout(10000) // 10 second timeout
    });
    
    if (!response.ok) {
      throw new Error(`API error: ${response.status}`);
    }
    
    const data = await response.json();
    
    if (!data.success) {
      throw new Error(data.error || 'Review failed');
    }
    
    return data.data;
    
  } catch (error) {
    if (error.name === 'AbortError') {
      throw new Error('Review request timed out. Please try again.');
    }
    
    if (error instanceof TypeError) {
      throw new Error('Network error. Please check your connection.');
    }
    
    throw error;
  }
}
```

#### LocalStorage Failures

```typescript
function safeLocalStorageOperation<T>(
  operation: () => T,
  fallback: T,
  errorMessage: string
): T {
  try {
    return operation();
  } catch (error) {
    console.error(errorMessage, error);
    
    // Check if quota exceeded
    if (error.name === 'QuotaExceededError') {
      console.warn('LocalStorage quota exceeded. Clearing old data...');
      clearOldAnalyticsData();
    }
    
    return fallback;
  }
}
```

#### Monaco Editor Failures

```typescript
function CodeEditor({ initialCode, onChange }: CodeEditorProps) {
  const [editorError, setEditorError] = useState<string | null>(null);
  
  const handleEditorDidMount = (editor: any, monaco: any) => {
    try {
      // Configure editor
      editor.updateOptions(editorOptions);
    } catch (error) {
      console.error('Editor initialization error:', error);
      setEditorError('Failed to initialize code editor');
    }
  };
  
  if (editorError) {
    return (
      <div className="flex items-center justify-center h-full bg-red-500/10">
        <p className="text-red-400">{editorError}</p>
      </div>
    );
  }
  
  return (
    <Editor
      defaultValue={initialCode}
      onChange={onChange}
      onMount={handleEditorDidMount}
      loading={<LoadingSpinner />}
    />
  );
}
```

### Server-Side Error Handling

#### API Route Error Boundaries

```typescript
export async function POST(request: NextRequest) {
  try {
    // Validate request
    const body = await request.json();
    
    if (!body.userCode || !body.ticketRequirements || !body.taskId) {
      return NextResponse.json(
        { 
          success: false, 
          error: 'Missing required fields: userCode, ticketRequirements, or taskId' 
        },
        { status: 400 }
      );
    }
    
    // Validate code length
    if (body.userCode.length > 50000) {
      return NextResponse.json(
        { 
          success: false, 
          error: 'Code submission too large (max 50KB)' 
        },
        { status: 413 }
      );
    }
    
    // Process review
    const reviewer = new AIReviewerService();
    const review = await reviewer.reviewCode(
      body.userCode,
      body.ticketRequirements,
      body.taskType || 'standard'
    );
    
    return NextResponse.json({
      success: true,
      data: review
    });
    
  } catch (error) {
    console.error('Review API error:', error);
    
    // Handle specific error types
    if (error.message?.includes('rate limit')) {
      return NextResponse.json(
        { 
          success: false, 
          error: 'Rate limit exceeded. Please try again in a moment.' 
        },
        { status: 429 }
      );
    }
    
    if (error.message?.includes('timeout')) {
      return NextResponse.json(
        { 
          success: false, 
          error: 'Review request timed out. Please try again.' 
        },
        { status: 504 }
      );
    }
    
    // Generic error response
    return NextResponse.json(
      { 
        success: false, 
        error: 'An unexpected error occurred. Please try again.' 
      },
      { status: 500 }
    );
  }
}
```

#### LLM API Error Handling

```typescript
class AIReviewerService {
  async reviewCode(
    userCode: string,
    requirements: string[],
    taskType: string
  ): Promise<ReviewResponse> {
    try {
      const prompt = this.buildReviewPrompt(userCode, requirements, taskType);
      const response = await this.llmClient.complete(prompt);
      const review = this.parseReviewResponse(response);
      
      return review;
      
    } catch (error) {
      console.error('AI review error:', error);
      
      // Return safe fallback response
      return {
        total_score: 0,
        rubric: {
          functional_correctness: 0,
          code_readability: 0,
          structure_modularity: 0,
          performance_efficiency: 0,
          security_practices: 0,
          error_handling: 0
        },
        status: 'REJECTED',
        feedback_summary: 'Unable to process review at this time. Please try again.',
        line_comments: [],
        diff_suggestions: []
      };
    }
  }
}
```

### User-Facing Error Messages

All error messages should be:
- **Clear**: Explain what went wrong in simple terms
- **Actionable**: Tell the user what they can do next
- **Non-technical**: Avoid exposing implementation details
- **Consistent**: Use the same tone and format throughout

Examples:
- ✅ "Review request timed out. Please try again."
- ✅ "Unable to save progress. Please check your browser settings."
- ❌ "Error: ECONNREFUSED at 127.0.0.1:3000"
- ❌ "Uncaught TypeError: Cannot read property 'map' of undefined"



## Testing Strategy

### Dual Testing Approach

The DevSim Dashboard requires both unit testing and property-based testing for comprehensive coverage:

- **Unit tests**: Verify specific examples, edge cases, and error conditions
- **Property tests**: Verify universal properties across all inputs

Both approaches are complementary and necessary. Unit tests catch concrete bugs in specific scenarios, while property tests verify general correctness across a wide range of inputs.

### Property-Based Testing

#### Library Selection

For TypeScript/JavaScript: **fast-check**

```bash
npm install --save-dev fast-check @types/fast-check
```

#### Configuration

Each property test should:
- Run a minimum of 100 iterations (due to randomization)
- Reference its design document property in a comment
- Use the tag format: `Feature: devsim-dashboard, Property {number}: {property_text}`

#### Example Property Tests

```typescript
import fc from 'fast-check';
import { describe, it, expect } from 'vitest';

describe('Task Card Properties', () => {
  // Feature: devsim-dashboard, Property 2: Task card information completeness
  it('should display ticket ID, difficulty, and XP for any task', () => {
    fc.assert(
      fc.property(
        fc.record({
          id: fc.string(),
          ticketId: fc.string(),
          difficulty: fc.constantFrom('Junior', 'Mid', 'Senior'),
          xpReward: fc.integer({ min: 50, max: 1000 }),
          status: fc.constantFrom('backlog', 'current_sprint', 'in_review', 'production')
        }),
        (task) => {
          const { container } = render(<TaskCard task={task} />);
          
          expect(container.textContent).toContain(task.ticketId);
          expect(container.textContent).toContain(task.difficulty);
          expect(container.textContent).toContain(task.xpReward.toString());
        }
      ),
      { numRuns: 100 }
    );
  });
  
  // Feature: devsim-dashboard, Property 4: Backlog task interaction prevention
  it('should prevent interaction for any task in backlog status', () => {
    fc.assert(
      fc.property(
        fc.record({
          id: fc.string(),
          ticketId: fc.string(),
          difficulty: fc.constantFrom('Junior', 'Mid', 'Senior'),
          xpReward: fc.integer({ min: 50, max: 1000 }),
          status: fc.constant('backlog')
        }),
        (task) => {
          const onClick = vi.fn();
          const { container } = render(<TaskCard task={task} onClick={onClick} />);
          
          const card = container.querySelector('[data-testid="task-card"]');
          fireEvent.click(card);
          
          expect(onClick).not.toHaveBeenCalled();
        }
      ),
      { numRuns: 100 }
    );
  });
});

describe('Rubric Scoring Properties', () => {
  // Feature: devsim-dashboard, Property 19: Total score calculation
  it('should calculate total score as sum of all category scores', () => {
    fc.assert(
      fc.property(
        fc.record({
          functional_correctness: fc.integer({ min: 0, max: 10 }),
          code_readability: fc.integer({ min: 0, max: 10 }),
          structure_modularity: fc.integer({ min: 0, max: 10 }),
          performance_efficiency: fc.integer({ min: 0, max: 10 }),
          security_practices: fc.integer({ min: 0, max: 10 }),
          error_handling: fc.integer({ min: 0, max: 10 })
        }),
        (rubric) => {
          const expectedTotal = Object.values(rubric).reduce((sum, score) => sum + score, 0);
          
          const review = {
            total_score: expectedTotal,
            rubric,
            status: 'APPROVED',
            feedback_summary: 'Test',
            line_comments: [],
            diff_suggestions: []
          };
          
          const calculatedTotal = Object.values(review.rubric).reduce((sum, score) => sum + score, 0);
          
          expect(review.total_score).toBe(calculatedTotal);
          expect(calculatedTotal).toBe(expectedTotal);
        }
      ),
      { numRuns: 100 }
    );
  });
  
  // Feature: devsim-dashboard, Property 21: Functional correctness rejection threshold
  it('should reject any review with functional correctness < 7', () => {
    fc.assert(
      fc.property(
        fc.record({
          functional_correctness: fc.integer({ min: 0, max: 6 }),
          code_readability: fc.integer({ min: 0, max: 10 }),
          structure_modularity: fc.integer({ min: 0, max: 10 }),
          performance_efficiency: fc.integer({ min: 0, max: 10 }),
          security_practices: fc.integer({ min: 6, max: 10 }),
          error_handling: fc.integer({ min: 0, max: 10 })
        }),
        (rubric) => {
          const status = determineReviewStatus(rubric, 'standard');
          expect(status).toBe('REJECTED');
        }
      ),
      { numRuns: 100 }
    );
  });
  
  // Feature: devsim-dashboard, Property 24: Approval threshold satisfaction
  it('should approve any review meeting all thresholds', () => {
    fc.assert(
      fc.property(
        fc.record({
          functional_correctness: fc.integer({ min: 7, max: 10 }),
          code_readability: fc.integer({ min: 0, max: 10 }),
          structure_modularity: fc.integer({ min: 0, max: 10 }),
          performance_efficiency: fc.integer({ min: 0, max: 10 }),
          security_practices: fc.integer({ min: 6, max: 10 }),
          error_handling: fc.integer({ min: 0, max: 10 })
        }).filter(rubric => {
          const total = Object.values(rubric).reduce((sum, score) => sum + score, 0);
          return total >= 30;
        }),
        (rubric) => {
          const status = determineReviewStatus(rubric, 'standard');
          expect(status).toBe('APPROVED');
        }
      ),
      { numRuns: 100 }
    );
  });
});

describe('Progress Tracking Properties', () => {
  // Feature: devsim-dashboard, Property 30: Completion percentage calculation
  it('should calculate completion percentage correctly for any task set', () => {
    fc.assert(
      fc.property(
        fc.array(fc.string(), { minLength: 1, maxLength: 20 }),
        fc.array(fc.string()),
        (allTaskIds, completedTaskIds) => {
          const completed = completedTaskIds.filter(id => allTaskIds.includes(id));
          const expectedPercentage = Math.round((completed.length / allTaskIds.length) * 100);
          
          const calculatedPercentage = calculateCompletionPercentage(completed.length, allTaskIds.length);
          
          expect(calculatedPercentage).toBe(expectedPercentage);
        }
      ),
      { numRuns: 100 }
    );
  });
  
  // Feature: devsim-dashboard, Property 34: Progress persistence round-trip
  it('should preserve progress data through save/load cycle', () => {
    fc.assert(
      fc.property(
        fc.record({
          completedTasks: fc.array(fc.string()),
          totalXP: fc.integer({ min: 0, max: 10000 }),
          currentLevel: fc.integer({ min: 1, max: 10 })
        }),
        (progress) => {
          saveProgressToStorage(progress);
          const loaded = loadProgressFromStorage();
          
          expect(loaded.completedTasks).toEqual(progress.completedTasks);
          expect(loaded.totalXP).toBe(progress.totalXP);
          expect(loaded.currentLevel).toBe(progress.currentLevel);
        }
      ),
      { numRuns: 100 }
    );
  });
});
```

### Unit Testing

#### Focus Areas for Unit Tests

Unit tests should focus on:
1. **Specific examples**: Concrete scenarios that demonstrate correct behavior
2. **Edge cases**: Boundary conditions and special cases
3. **Error conditions**: How the system handles invalid inputs and failures
4. **Integration points**: How components work together

#### Example Unit Tests

```typescript
import { describe, it, expect, vi } from 'vitest';
import { render, fireEvent, waitFor } from '@testing-library/react';

describe('Dashboard Page', () => {
  it('should display sidebar and Kanban board on /dashboard', () => {
    const { getByTestId } = render(<DashboardPage />);
    
    expect(getByTestId('sidebar')).toBeInTheDocument();
    expect(getByTestId('kanban-board')).toBeInTheDocument();
  });
  
  it('should display progress circle with completion percentage', () => {
    const { getByTestId } = render(<DashboardPage />);
    
    const progressCircle = getByTestId('progress-circle');
    expect(progressCircle).toBeInTheDocument();
  });
});

describe('Editor Page', () => {
  it('should display split-view layout with 30/70 width allocation', () => {
    const { getByTestId } = render(<EditorPage params={{ taskId: 'task-001' }} />);
    
    const ticketPanel = getByTestId('ticket-panel');
    const editorPanel = getByTestId('editor-panel');
    
    expect(ticketPanel).toHaveStyle({ width: '30%' });
    expect(editorPanel).toHaveStyle({ width: '70%' });
  });
  
  it('should display Submit PR button with gradient purple styling', () => {
    const { getByText } = render(<EditorPage params={{ taskId: 'task-001' }} />);
    
    const submitButton = getByText('Submit PR');
    expect(submitButton).toHaveClass('bg-gradient-to-r', 'from-purple-500', 'to-pink-500');
  });
});

describe('Review Modal', () => {
  it('should display confetti animation when status is APPROVED', async () => {
    const approvedReview = {
      total_score: 50,
      rubric: {
        functional_correctness: 9,
        code_readability: 8,
        structure_modularity: 8,
        performance_efficiency: 8,
        security_practices: 9,
        error_handling: 8
      },
      status: 'APPROVED',
      feedback_summary: 'Great work!',
      line_comments: [],
      diff_suggestions: []
    };
    
    const { getByTestId } = render(<ReviewModal isOpen={true} review={approvedReview} />);
    
    await waitFor(() => {
      expect(getByTestId('confetti-animation')).toBeInTheDocument();
    });
  });
  
  it('should display feedback modal when status is REJECTED', () => {
    const rejectedReview = {
      total_score: 25,
      rubric: {
        functional_correctness: 5,
        code_readability: 4,
        structure_modularity: 4,
        performance_efficiency: 4,
        security_practices: 4,
        error_handling: 4
      },
      status: 'REJECTED',
      feedback_summary: 'Code needs improvement',
      line_comments: [
        { line_number: 10, comment: 'Missing error handling', severity: 'error' }
      ],
      diff_suggestions: []
    };
    
    const { getByText } = render(<ReviewModal isOpen={true} review={rejectedReview} />);
    
    expect(getByText('Code needs improvement')).toBeInTheDocument();
    expect(getByText('Missing error handling')).toBeInTheDocument();
  });
});

describe('Challenge Data', () => {
  it('should include Level 1 challenge: Fix the React Counter', () => {
    const level1 = challenges.find(c => c.level === 1);
    
    expect(level1).toBeDefined();
    expect(level1.task.title).toBe('Fix the React Counter');
    expect(level1.task.buggyCode).toContain('useState');
  });
  
  it('should include Level 2 challenge: Secure the API Endpoint', () => {
    const level2 = challenges.find(c => c.level === 2);
    
    expect(level2).toBeDefined();
    expect(level2.task.title).toBe('Secure the API Endpoint');
    expect(level2.task.requirements).toContain('Move API key to environment variable');
  });
  
  it('should include Level 3 challenge: Optimize Search', () => {
    const level3 = challenges.find(c => c.level === 3);
    
    expect(level3).toBeDefined();
    expect(level3.task.title).toBe('Optimize Search Algorithm');
    expect(level3.task.type).toBe('production_incident');
  });
});

describe('API Error Handling', () => {
  it('should return 400 when userCode is missing', async () => {
    const response = await POST(
      new NextRequest('http://localhost:3000/api/review', {
        method: 'POST',
        body: JSON.stringify({
          ticketRequirements: ['requirement 1'],
          taskId: 'task-001'
        })
      })
    );
    
    expect(response.status).toBe(400);
    const data = await response.json();
    expect(data.success).toBe(false);
    expect(data.error).toContain('Missing required fields');
  });
  
  it('should handle malformed JSON gracefully', async () => {
    const response = await POST(
      new NextRequest('http://localhost:3000/api/review', {
        method: 'POST',
        body: 'invalid json'
      })
    );
    
    expect(response.status).toBe(500);
    const data = await response.json();
    expect(data.success).toBe(false);
  });
});
```

### Test Coverage Goals

- **Unit Test Coverage**: Minimum 80% code coverage
- **Property Test Coverage**: All correctness properties implemented
- **Integration Test Coverage**: All critical user flows tested
- **E2E Test Coverage**: Happy path and error scenarios

### Continuous Integration

Tests should run automatically on:
- Every commit to main branch
- Every pull request
- Pre-deployment to production

```yaml
# .github/workflows/test.yml
name: Test Suite

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm run test:unit
      - run: npm run test:property
      - run: npm run test:integration
```

### Testing Best Practices

1. **Keep tests focused**: Each test should verify one specific behavior
2. **Use descriptive names**: Test names should clearly state what they verify
3. **Avoid test interdependence**: Tests should not rely on execution order
4. **Mock external dependencies**: Use mocks for API calls and external services
5. **Test behavior, not implementation**: Focus on what the code does, not how
6. **Maintain test data**: Keep test fixtures organized and maintainable
7. **Run tests frequently**: Run tests during development, not just in CI



## Implementation Considerations

### Performance Optimization

#### Code Splitting

```typescript
// Lazy load heavy components
const MonacoEditor = dynamic(() => import('@monaco-editor/react'), {
  ssr: false,
  loading: () => <LoadingSpinner />
});

const AnalyticsGraph = dynamic(() => import('@/components/analytics/ScoreGraph'), {
  loading: () => <LoadingSpinner />
});
```

#### Memoization

```typescript
// Memoize expensive calculations
const completionPercentage = useMemo(() => {
  return Math.round((completedTasks.length / totalTasks) * 100);
}, [completedTasks.length, totalTasks]);

// Memoize component renders
const TaskCard = memo(({ task, onClick }: TaskCardProps) => {
  // Component implementation
});
```

#### Debouncing Editor Changes

```typescript
const debouncedOnChange = useMemo(
  () => debounce((value: string) => {
    setCode(value);
  }, 300),
  []
);
```

### Security Considerations

#### Input Sanitization

```typescript
function sanitizeCode(code: string): string {
  // Remove potentially dangerous patterns
  const sanitized = code
    .replace(/eval\(/g, '/* eval( */')
    .replace(/Function\(/g, '/* Function( */');
  
  return sanitized;
}
```

#### API Rate Limiting

```typescript
// Implement rate limiting middleware
const rateLimiter = new RateLimiter({
  windowMs: 60000, // 1 minute
  maxRequests: 10 // 10 requests per minute
});

export async function POST(request: NextRequest) {
  const clientId = request.headers.get('x-forwarded-for') || 'unknown';
  
  if (!rateLimiter.checkLimit(clientId)) {
    return NextResponse.json(
      { success: false, error: 'Rate limit exceeded' },
      { status: 429 }
    );
  }
  
  // Process request
}
```

#### Environment Variable Validation

```typescript
// Validate required environment variables at startup
function validateEnvironment() {
  const required = ['OPENAI_API_KEY', 'AWS_REGION'];
  
  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
}
```

### Accessibility

#### Keyboard Navigation

```typescript
function TaskCard({ task, onClick }: TaskCardProps) {
  const handleKeyPress = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      onClick?.();
    }
  };
  
  return (
    <div
      role="button"
      tabIndex={task.status === 'backlog' ? -1 : 0}
      onClick={onClick}
      onKeyPress={handleKeyPress}
      aria-label={`Task ${task.ticketId}: ${task.title}`}
    >
      {/* Card content */}
    </div>
  );
}
```

#### ARIA Labels

```typescript
<button
  aria-label="Submit code for review"
  aria-describedby="submit-help-text"
>
  Submit PR
</button>
<span id="submit-help-text" className="sr-only">
  Your code will be reviewed by the AI agent
</span>
```

#### Screen Reader Support

```typescript
<div role="status" aria-live="polite" aria-atomic="true">
  {reviewStatus === 'loading' && 'Reviewing your code...'}
  {reviewStatus === 'approved' && 'Code approved! Great work!'}
  {reviewStatus === 'rejected' && 'Code needs improvement. Review feedback below.'}
</div>
```

### Responsive Design

#### Breakpoints

```typescript
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      'sm': '640px',
      'md': '768px',
      'lg': '1024px',
      'xl': '1280px',
      '2xl': '1536px',
    }
  }
}
```

#### Mobile Layout Adjustments

```typescript
// Stack panels vertically on mobile
<div className="flex flex-col lg:flex-row">
  <div className="w-full lg:w-[30%]">
    <TicketPanel task={task} />
  </div>
  <div className="w-full lg:w-[70%]">
    <CodeEditor code={code} />
  </div>
</div>
```

### Monitoring and Logging

#### Client-Side Error Tracking

```typescript
window.addEventListener('error', (event) => {
  console.error('Global error:', event.error);
  
  // Send to monitoring service
  trackError({
    message: event.error.message,
    stack: event.error.stack,
    url: window.location.href
  });
});
```

#### API Performance Monitoring

```typescript
export async function POST(request: NextRequest) {
  const startTime = Date.now();
  
  try {
    // Process request
    const result = await processReview(request);
    
    const duration = Date.now() - startTime;
    console.log(`Review completed in ${duration}ms`);
    
    return result;
  } catch (error) {
    const duration = Date.now() - startTime;
    console.error(`Review failed after ${duration}ms:`, error);
    throw error;
  }
}
```

### Future Enhancements

1. **Multi-language Support**: Add challenges for Python, Java, Go
2. **Collaborative Mode**: Allow users to share solutions and compare approaches
3. **Leaderboard**: Track top performers and fastest completion times
4. **Custom Challenges**: Allow users to create and share their own challenges
5. **Video Tutorials**: Integrate video explanations for complex concepts
6. **Code Playback**: Show how the user's code evolved during the session
7. **AI Pair Programming**: Real-time suggestions as the user types
8. **Achievement System**: Badges and milestones for learning progress
9. **Export Progress**: Download certificates or progress reports
10. **Integration with GitHub**: Submit solutions as actual PRs to a repository

