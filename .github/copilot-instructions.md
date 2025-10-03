# GitHub Copilot Instructions for MCP Inspector

## Project Overview
You're working on **MCP Inspector Pro**, an enterprise-grade web application for testing Model Context Protocol (MCP) servers. This is a production system used by DevOps engineers.

## Critical Requirements
- **Node.js**: 22.7.5+ (MUST use, package requires it)
- **TypeScript**: Strict mode, explicit types
- **Framework**: Next.js 14 with App Router
- **Styling**: Tailwind CSS only
- **Testing**: Jest + React Testing Library
- **Deployment**: Vercel via GitHub Actions

## Code Style

### Always Use TypeScript
```typescript
// ✅ GOOD
interface CommandConfig {
  serverUrl: string;
  method: 'tools/list' | 'tools/call';
}

function executeCommand(config: CommandConfig): Promise<Response> {
  // implementation
}

// ❌ AVOID
function executeCommand(config: any) {
  // implementation
}
```

### React Components
```typescript
// ✅ GOOD - Functional with explicit types
'use client'

interface ButtonProps {
  onClick: () => void;
  children: React.ReactNode;
  disabled?: boolean;
}

export const Button: React.FC<ButtonProps> = ({ 
  onClick, 
  children, 
  disabled = false 
}) => {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className="px-4 py-2 bg-blue-600 text-white rounded"
    >
      {children}
    </button>
  );
};

// ❌ AVOID - Class components
class Button extends React.Component {
  render() {
    return <button>Click</button>;
  }
}
```

### Error Handling
```typescript
// ✅ GOOD - Comprehensive error handling
async function fetchData(): Promise<Data> {
  try {
    const response = await fetch(url);
    
    if (!response.ok) {
      throw new NetworkError(`HTTP ${response.status}: ${response.statusText}`);
    }
    
    const data = await response.json();
    return validateData(data);
    
  } catch (error) {
    if (error instanceof NetworkError) {
      logError('Network request failed', { error });
      throw error;
    }
    
    throw new MCPError('Unexpected error', { cause: error });
  }
}

// ❌ AVOID - Silent failures
async function fetchData() {
  try {
    const response = await fetch(url);
    return response.json();
  } catch (e) {
    return null; // Silent failure!
  }
}
```

## Specific Patterns

### API Routes (Next.js 14)
```typescript
// app/api/execute/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

const RequestSchema = z.object({
  serverUrl: z.string().url(),
  method: z.enum(['tools/list', 'tools/call']),
});

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const validated = RequestSchema.parse(body);
    
    const result = await executeCommand(validated);
    
    return NextResponse.json({ success: true, data: result });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { success: false, error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }
    
    return NextResponse.json(
      { success: false, error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Custom Hooks
```typescript
// lib/hooks/useMCPCommand.ts
import { useState, useCallback } from 'react';

interface UseMCPCommandResult {
  data: MCPResponse | null;
  loading: boolean;
  error: Error | null;
  execute: () => Promise<void>;
}

export function useMCPCommand(config: MCPConfig): UseMCPCommandResult {
  const [data, setData] = useState<MCPResponse | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const execute = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await executeMCPCommand(config);
      setData(result);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  }, [config]);

  return { data, loading, error, execute };
}
```

### State Management
```typescript
// Use useReducer for complex state
type State = {
  config: MCPConfig;
  output: string[];
  history: Command[];
  isExecuting: boolean;
};

type Action =
  | { type: 'SET_CONFIG'; payload: MCPConfig }
  | { type: 'ADD_OUTPUT'; payload: string }
  | { type: 'START_EXECUTION' }
  | { type: 'FINISH_EXECUTION' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_CONFIG':
      return { ...state, config: action.payload };
    case 'ADD_OUTPUT':
      return { 
        ...state, 
        output: [...state.output, action.payload] 
      };
    case 'START_EXECUTION':
      return { ...state, isExecuting: true };
    case 'FINISH_EXECUTION':
      return { ...state, isExecuting: false };
    default:
      return state;
  }
}
```

## Styling Guidelines

### Tailwind CSS Only
```typescript
// ✅ GOOD
<button className="
  px-4 py-2 
  bg-blue-600 hover:bg-blue-700 
  text-white font-medium 
  rounded-lg shadow-sm
  transition-colors duration-200
  disabled:opacity-50 disabled:cursor-not-allowed
  focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
">
  Execute
</button>

// ❌ AVOID
<button style={{ 
  padding: '8px 16px', 
  backgroundColor: '#3b82f6' 
}}>
  Execute
</button>
```

### Responsive Design
```typescript
// Mobile-first approach
<div className="
  flex flex-col gap-4
  md:flex-row md:gap-6
  lg:gap-8
">
  <div className="w-full md:w-1/2 lg:w-1/3">
    {/* Content */}
  </div>
</div>
```

## Testing

### Component Tests
```typescript
// __tests__/CommandButton.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { CommandButton } from '../CommandButton';

describe('CommandButton', () => {
  it('executes command when clicked', async () => {
    const mockExecute = jest.fn().mockResolvedValue({ success: true });
    
    render(<CommandButton onExecute={mockExecute} />);
    
    const button = screen.getByRole('button', { name: /execute/i });
    fireEvent.click(button);
    
    await waitFor(() => {
      expect(mockExecute).toHaveBeenCalledTimes(1);
    });
  });

  it('shows loading state during execution', async () => {
    const mockExecute = jest.fn(
      () => new Promise(resolve => setTimeout(resolve, 100))
    );
    
    render(<CommandButton onExecute={mockExecute} />);
    
    const button = screen.getByRole('button');
    fireEvent.click(button);
    
    expect(button).toHaveTextContent('Executing...');
    expect(button).toBeDisabled();
  });
});
```

## Accessibility

### ARIA Labels
```typescript
// ✅ GOOD
<button
  onClick={handleExecute}
  disabled={isLoading}
  aria-label="Execute MCP command"
  aria-busy={isLoading}
>
  <PlayIcon aria-hidden="true" />
  Execute
</button>

<input
  type="text"
  id="server-url"
  aria-label="Server URL"
  aria-required="true"
  aria-invalid={!!error}
  aria-describedby={error ? 'url-error' : undefined}
/>
{error && (
  <span id="url-error" role="alert">
    {error}
  </span>
)}
```

## Security

### Input Validation
```typescript
import { z } from 'zod';
import DOMPurify from 'isomorphic-dompurify';

// Validate all inputs
const ConfigSchema = z.object({
  serverUrl: z.string().url().refine(
    url => new URL(url).protocol === 'https:',
    'Only HTTPS URLs allowed'
  ),
  method: z.enum(['tools/list', 'tools/call']),
});

// Sanitize HTML
function sanitizeOutput(html: string): string {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'code', 'pre'],
  });
}
```

### Environment Variables
```typescript
// Only expose variables with NEXT_PUBLIC_ prefix
const apiUrl = process.env.NEXT_PUBLIC_API_URL;

// Server-side only (never exposed to client)
const secretKey = process.env.SECRET_KEY;
```

## Performance

### Code Splitting
```typescript
import dynamic from 'next/dynamic';

// Lazy load heavy components
const CodeEditor = dynamic(() => import('./CodeEditor'), {
  loading: () => <Skeleton />,
  ssr: false,
});

const ChartComponent = dynamic(() => import('./Chart'), {
  loading: () => <div>Loading chart...</div>,
});
```

### Memoization
```typescript
import { memo, useMemo, useCallback } from 'react';

// Memoize expensive components
export const ExpensiveComponent = memo(({ data }) => {
  const processedData = useMemo(() => {
    return data.map(item => expensiveOperation(item));
  }, [data]);

  const handleClick = useCallback((id: string) => {
    // Handle click
  }, []);

  return <div>{/* render */}</div>;
});
```

## Common Mistakes to Avoid

### ❌ Don't
- Use `any` type
- Ignore TypeScript errors
- Skip error handling
- Use inline styles
- Mutate state directly
- Forget accessibility
- Hardcode secrets
- Use deprecated APIs

### ✅ Do
- Use explicit types
- Handle all errors
- Use Tailwind CSS
- Immutable state updates
- Include ARIA labels
- Use environment variables
- Keep dependencies updated
- Write tests

## Git Commit Format
```bash
feat(ui): add command history panel
fix(api): handle network timeout errors
docs(readme): update installation steps
refactor(hooks): simplify useMCPCommand logic
test(api): add integration tests for execute endpoint
chore(deps): upgrade Next.js to 14.0.4
```

## Quick Checklist Before Suggesting Code
- [ ] TypeScript types are explicit
- [ ] Error handling included
- [ ] Tailwind CSS used (no inline styles)
- [ ] Accessible (ARIA labels, semantic HTML)
- [ ] Mobile-responsive
- [ ] Input validated
- [ ] Tests included (for new features)
- [ ] JSDoc comments for public APIs
- [ ] No hardcoded secrets
- [ ] Follows project structure

## Remember
When suggesting code:
1. Make it production-ready
2. Include error handling
3. Use TypeScript properly
4. Follow accessibility guidelines
5. Optimize for mobile
6. Add comments for complex logic
7. Consider security implications
8. Write testable code

This is a production system - quality over speed!
