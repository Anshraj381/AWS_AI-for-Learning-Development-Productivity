# Requirements Document

## Introduction

The DevSim Dashboard is an AI Junior Developer Simulator that provides an interactive learning environment where users practice fixing broken code under the supervision of an AI Senior Developer Agent. The system features a glassmorphism-styled dashboard and integrated code editor, simulating a realistic software development workflow with ticket management, code review, and progression tracking.

## Glossary

- **System**: The DevSim Dashboard application
- **User**: A developer using the simulator to practice coding skills
- **AI_Agent**: The simulated Senior Developer that reviews code submissions
- **Task**: A coding challenge presented as a Jira-style ticket
- **PR_Submission**: User's code submission for review
- **Kanban_Board**: The visual task management interface on the dashboard
- **Workstation**: The integrated development environment with ticket details and code editor
- **Review_Response**: The AI Agent's feedback on a code submission
- **XP**: Experience points awarded for completing tasks
- **Glassmorphism**: The visual design style using frosted glass effects with backdrop blur
- **Rubric**: A structured scoring system evaluating code across multiple categories
- **Total_Score**: The sum of all rubric category scores (maximum 60)
- **Production_Incident**: An advanced challenge simulating performance issues in production code

## Requirements

### Requirement 1: Dashboard Navigation and Layout

**User Story:** As a user, I want to view a dashboard with my tasks organized in a Kanban board, so that I can track my progress and select tasks to work on.

#### Acceptance Criteria

1. WHEN a user navigates to /dashboard, THE System SHALL display a sidebar on the left and a Kanban board in the main area
2. THE Kanban_Board SHALL display four columns: "Backlog", "Current Sprint", "In Review", and "Production"
3. WHEN displaying task cards, THE System SHALL show the ticket ID, difficulty level, and reward XP for each task
4. THE System SHALL display a progress circle chart showing internship completion percentage
5. THE System SHALL apply glassmorphism styling with bg-slate-950 background and bg-white/5 with backdrop-blur-xl for cards

### Requirement 2: Task Card Management

**User Story:** As a user, I want to see task cards with relevant information, so that I can understand the difficulty and rewards before starting a task.

#### Acceptance Criteria

1. WHEN a task card is displayed, THE System SHALL show the ticket ID in a visible format
2. WHEN a task card is displayed, THE System SHALL indicate the difficulty level as Junior, Mid, or Senior
3. WHEN a task card is displayed, THE System SHALL show the XP reward amount
4. WHEN a task is in the Backlog column, THE System SHALL prevent user interaction with locked tasks
5. WHEN a task is in the Current Sprint column, THE System SHALL allow users to click and navigate to the workstation

### Requirement 3: Workstation Interface

**User Story:** As a user, I want to work on tasks in a split-view editor, so that I can see the requirements while writing code.

#### Acceptance Criteria

1. WHEN a user navigates to /editor/[taskId], THE System SHALL display a split-view layout
2. THE System SHALL allocate 30% of the viewport width to the left panel displaying the ticket
3. THE System SHALL allocate 70% of the viewport width to the right panel displaying the Monaco editor
4. WHEN displaying the ticket panel, THE System SHALL show the context, requirements, and API documentation
5. THE System SHALL pre-load the Monaco editor with the buggy code specific to the selected task
6. THE System SHALL display a floating "Submit PR" button with gradient purple styling

### Requirement 4: Code Editor Functionality

**User Story:** As a user, I want to edit code in a professional IDE-style editor, so that I can fix bugs efficiently with syntax highlighting and code completion.

#### Acceptance Criteria

1. THE System SHALL integrate @monaco-editor/react for the code editor component
2. WHEN the editor loads, THE System SHALL apply JetBrains Mono font for code display
3. THE System SHALL enable syntax highlighting appropriate to the code language
4. THE System SHALL allow users to edit the pre-loaded code
5. WHEN a user clicks the Submit PR button, THE System SHALL capture the current editor content

### Requirement 5: AI Code Review Processing

**User Story:** As a user, I want my code submissions reviewed by an AI agent, so that I can receive feedback on my solutions.

#### Acceptance Criteria

1. WHEN a user submits a PR, THE System SHALL send the user code and ticket requirements to the AI_Agent via /api/review
2. THE AI_Agent SHALL analyze the code against the ticket requirements
3. THE AI_Agent SHALL return a JSON response containing total_score, rubric, status, feedback_summary, line_comments, and diff_suggestions
4. THE System SHALL process the Review_Response within 10 seconds of submission
5. IF the AI_Agent returns REJECTED status, THEN THE System SHALL display a code review modal with detailed feedback
6. IF the AI_Agent returns APPROVED status, THEN THE System SHALL display a confetti animation and redirect to the dashboard
7. THE System SHALL use AWS Bedrock or OpenAI API for LLM integration

### Requirement 6: AI Agent Persona and Behavior

**User Story:** As a user, I want to receive professional and constructive code reviews, so that I can learn best practices and improve my coding skills.

#### Acceptance Criteria

1. THE AI_Agent SHALL adopt the persona of "Sarah Chen, Senior Staff Engineer"
2. WHEN reviewing code, THE AI_Agent SHALL evaluate code quality beyond functional correctness
3. THE AI_Agent SHALL reject code with poor variable naming, missing comments, or messy structure even if functionally correct
4. WHEN providing feedback, THE AI_Agent SHALL explain why code is wrong by highlighting security, performance, or clean code issues
5. THE AI_Agent SHALL maintain a professional, brief, and constructive tone in all feedback
6. THE AI_Agent SHALL provide guidance and hints rather than complete working solutions
7. THE AI_Agent SHALL NOT provide full implementations that solve the entire problem

### Requirement 6A: AI Reviewer Rubric System

**User Story:** As a user, I want my code evaluated using a structured rubric, so that I can understand specific areas where my code needs improvement.

#### Acceptance Criteria

1. THE AI_Agent SHALL evaluate code using six scoring categories: Functional Correctness, Code Readability, Structure and Modularity, Performance Efficiency, Security Best Practices, and Error Handling
2. WHEN scoring each category, THE AI_Agent SHALL assign a score from 0 to 10
3. THE AI_Agent SHALL calculate the Total_Score as the sum of all category scores (maximum 60)
4. IF Functional Correctness score is less than 7, THEN THE AI_Agent SHALL return REJECTED status
5. IF Security Best Practices score is less than 6, THEN THE AI_Agent SHALL return REJECTED status
6. IF Total_Score is less than 30, THEN THE AI_Agent SHALL return REJECTED status
7. WHEN all thresholds are met, THE AI_Agent SHALL return APPROVED status
8. THE System SHALL display the rubric scores in the code review modal

### Requirement 7: Challenge Data Management

**User Story:** As a user, I want to work through progressively difficult challenges, so that I can develop my skills incrementally.

#### Acceptance Criteria

1. THE System SHALL provide three hardcoded challenge levels in lib/challenges.ts
2. THE System SHALL include Level 1 challenge: "Fix the React Counter" with a missing onClick handler
3. THE System SHALL include Level 2 challenge: "Secure the API Endpoint" requiring environment variable usage
4. THE System SHALL include Level 3 challenge: "Optimize Search" requiring algorithm optimization from O(n²) to O(n)
5. WHEN a challenge is loaded, THE System SHALL provide the buggy code, context, requirements, and expected solution criteria

### Requirement 8: Progress Tracking and Rewards

**User Story:** As a user, I want to track my progress and earn rewards, so that I feel motivated to complete more challenges.

#### Acceptance Criteria

1. WHEN a user completes a task with APPROVED status, THE System SHALL award the specified XP amount
2. THE System SHALL update the internship completion percentage based on completed tasks
3. WHEN the AI_Agent approves code, THE System SHALL move the task from "In Review" to "Production" column
4. WHEN displaying progress, THE System SHALL show the percentage in a circular progress chart
5. THE System SHALL persist progress data across user sessions

### Requirement 9: Visual Design System

**User Story:** As a user, I want an aesthetically pleasing cyberpunk-themed interface, so that the learning experience is engaging and modern.

#### Acceptance Criteria

1. THE System SHALL use bg-slate-950 as the primary background color
2. THE System SHALL apply glassmorphism effects using bg-white/5 with backdrop-blur-xl for card elements
3. THE System SHALL use neon purple (#a855f7) for accent elements and buttons
4. THE System SHALL use success green (#22c55e) for positive feedback and completed states
5. THE System SHALL use Inter font for UI elements and JetBrains Mono for code display
6. THE System SHALL integrate lucide-react for iconography

### Requirement 10: Task State Transitions

**User Story:** As a user, I want tasks to move through workflow states automatically, so that I can see my progress reflected in the Kanban board.

#### Acceptance Criteria

1. WHEN a user clicks a task in "Current Sprint", THE System SHALL navigate to the workstation
2. WHEN a user submits a PR, THE System SHALL move the task to "In Review" column
3. WHEN the AI_Agent approves code, THE System SHALL move the task to "Production" column
4. WHEN the AI_Agent rejects code, THE System SHALL return the task to "Current Sprint" column
5. THE System SHALL prevent tasks in "Backlog" from being moved until prerequisites are met

### Requirement 11: API Route Implementation

**User Story:** As a developer, I want a well-structured API for code review, so that the AI agent can process submissions efficiently.

#### Acceptance Criteria

1. THE System SHALL implement the review endpoint at app/api/review/route.ts
2. WHEN the endpoint receives a request, THE System SHALL validate the presence of userCode and ticketRequirements
3. THE System SHALL integrate with OpenAI SDK or Gemini API for LLM processing
4. THE System SHALL format the AI response as JSON with status, feedback_summary, line_comments, and diff_suggestions fields
5. IF the API request fails, THEN THE System SHALL return an error response with appropriate HTTP status code

### Requirement 11A: JSON Response Format Structure

**User Story:** As a developer, I want a standardized JSON response format from the AI reviewer, so that the frontend can consistently display review results.

#### Acceptance Criteria

1. THE Review_Response SHALL include a total_score field containing a number from 0 to 60
2. THE Review_Response SHALL include a rubric object with six category scores: functional_correctness, code_readability, structure_modularity, performance_efficiency, security_practices, and error_handling
3. THE Review_Response SHALL include a status field with value "APPROVED" or "REJECTED"
4. THE Review_Response SHALL include a feedback_summary field containing a string with overall feedback
5. THE Review_Response SHALL include a line_comments array containing objects with line_number and comment fields
6. THE Review_Response SHALL include a diff_suggestions array containing suggested code improvements
7. WHEN the response is malformed, THE System SHALL handle the error gracefully and display a user-friendly message

### Requirement 12: Responsive Layout and Animations

**User Story:** As a user, I want smooth animations and responsive layouts, so that the interface feels polished and professional.

#### Acceptance Criteria

1. THE System SHALL integrate Framer Motion for animations
2. WHEN a task card is moved between columns, THE System SHALL animate the transition
3. WHEN code is approved, THE System SHALL display a confetti animation using a celebration library
4. THE System SHALL ensure the split-view layout adapts to different screen sizes
5. WHEN hovering over interactive elements, THE System SHALL provide visual feedback with smooth transitions

### Requirement 13: Production Incident Mode

**User Story:** As an advanced user, I want to work on production incident challenges, so that I can practice optimizing real-world performance issues.

#### Acceptance Criteria

1. THE System SHALL provide a Production_Incident challenge type as an advanced feature
2. WHEN a Production_Incident is loaded, THE System SHALL present code with performance issues requiring optimization
3. THE System SHALL include context describing the production symptoms (e.g., slow response times, high memory usage)
4. THE AI_Agent SHALL evaluate Production_Incident submissions with emphasis on Performance Efficiency rubric scores
5. WHEN reviewing Production_Incident code, THE AI_Agent SHALL require Performance Efficiency score of at least 8 for approval

### Requirement 14: Responsible AI and Data Privacy

**User Story:** As a user, I want the AI to guide my learning without solving problems for me, and I want my data to remain private.

#### Acceptance Criteria

1. THE AI_Agent SHALL NOT provide complete working solutions to challenges
2. WHEN providing feedback, THE AI_Agent SHALL offer hints, guidance, and explanations rather than full code implementations
3. THE System SHALL NOT store personally identifiable information persistently
4. THE System SHALL process code submissions transiently without long-term storage of user code
5. THE System SHALL use session-based state management for user progress without requiring authentication

### Requirement 15: Technical Architecture Standards

**User Story:** As a developer, I want the codebase to follow clean architecture principles, so that the system is maintainable and scalable.

#### Acceptance Criteria

1. THE System SHALL implement clean architecture with clear separation between UI, business logic, and data layers
2. THE System SHALL use modular components with single responsibility principle
3. THE System SHALL implement scalable state management using React Context or a state management library
4. THE System SHALL organize code into logical directories (components, lib, app, types)
5. THE System SHALL use TypeScript for type safety across all modules

### Requirement 16: Learning Analytics and Growth Tracking

**User Story:** As a user, I want to see how my code quality improves over time, so that I can track my learning progress and identify areas for improvement.

#### Acceptance Criteria

1. THE System SHALL track historical Total_Score for each task attempt
2. THE System SHALL display a score progression graph showing improvement over time
3. WHEN a user reattempts a task, THE System SHALL calculate the improvement delta between attempts
4. THE System SHALL identify and highlight the user's strongest rubric category based on historical scores
5. THE System SHALL identify and highlight the user's weakest rubric category based on historical scores
6. THE System SHALL persist analytics data across user sessions
