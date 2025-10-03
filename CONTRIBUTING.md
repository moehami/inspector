# Contributing to MCP Inspector Pro

Thank you for your interest in contributing! This document provides guidelines and instructions for contributing to the project.

## 📋 Table of Contents
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Code Standards](#code-standards)
- [Making Changes](#making-changes)
- [Testing](#testing)
- [Submitting Changes](#submitting-changes)
- [AI-Assisted Development](#ai-assisted-development)

## 🚀 Getting Started

### Prerequisites
- **Node.js**: 22.7.5 or higher (critical!)
- **npm**: 10.0.0 or higher
- **Git**: Latest version
- **Editor**: VS Code, Cursor, or similar with TypeScript support

### Fork and Clone
```bash
# Fork the repository on GitHub, then clone your fork
git clone https://github.com/YOUR_USERNAME/mcp-inspector.git
cd mcp-inspector

# Add upstream remote
git remote add upstream https://github.com/ORIGINAL_OWNER/mcp-inspector.git
```

## 💻 Development Setup

### 1. Install Node.js 22
```bash
# Using nvm (recommended)
nvm install 22.7.5
nvm use 22.7.5
nvm alias default 22.7.5

# Verify installation
node --version  # Should show v22.7.5 or higher
npm --version   # Should show 10.x.x or higher
```

### 2. Install Dependencies
```bash
npm install --legacy-peer-deps
```

### 3. Environment Setup
```bash
# Copy example environment file
cp .env.example .env.local

# Add your configuration
# NEXT_PUBLIC_API_URL=http://localhost:3000
```

### 4. Start Development Server
```bash
npm run dev

# Open http://localhost:3000
```

### 5. Verify Setup
```bash
# Type checking
npm run type-check

# Linting
npm run lint

# Tests
npm test

# Build
npm run build
```

## 📝 Code Standards

### TypeScript
```typescript
// ✅ DO: Explicit types, clear interfaces
interface MCPServerConfig {
  serverUrl: string;
  method: 'tools/list' | 'tools/call' | 'resources/list';
  transport: 'stdio' | 'http' | 'sse';
  headers?: Record<string, string>;
  timeout?: number;
}

function executeCommand(config: MCPServerConfig): Promise<MCPResponse> {
  // Implementation
}

// ❌ DON'T: Implicit any, unclear types
function executeCommand(config: any) {
  // Implementation
}
```

### React Components
```typescript
// ✅ DO: Functional components with TypeScript
'use client'

import React, { useState, useCallback } from 'react';
import { Play } from 'lucide-react';

interface CommandButtonProps {
  onExecute: () => Promise<void>;
  disabled?: boolean;
}

export const CommandButton: React.FC<CommandButtonProps> = ({ 
  onExecute, 
  disabled = false 
}) => {
  const [isLoading, setIsLoading] = useState(false);

  const handleClick = useCallback(async () => {
    setIsLoading(true);
    try {
      await onExecute();
    } catch (error) {
      console.error('Execution failed:', error);
    } finally {
      setIsLoading(false);
    }
  }, [onExecute]);

  return (
    <button
      onClick={handleClick}
      disabled={disabled || isLoading}
      className="btn-primary"
      aria-label="Execute command"
      aria-busy={isLoading}
    >
      <Play className="w-4 h-4" />
      {isLoading ? 'Executing...' : 'Execute'}
    </button>
  );
};

// ❌ DON'T: Class components, missing types
class CommandButton extends React.Component {
  render() {
    return <button onClick={this.props.onClick}>Execute</button>;
  }
}
```

### Styling
```typescript
// ✅ DO: Tailwind CSS with accessibility
<button className="
  px-4 py-2 min-h-[44px]
  bg-blue-600 hover:bg-blue-700
  text-white font-medium
  rounded-lg shadow-sm
  transition-colors duration-200
  focus:outline-none focus:ring-2 focus:ring-blue-500
  disabled:opacity-50 disabled:cursor-not-allowed
  touch-manipulation
">
  Execute
</button>

// ❌ DON'T: Inline styles
<button style={{ padding: '8px 16px', backgroundColor: 'blue' }}>
  Execute
</button>
```

### Error Handling
```typescript
// ✅ DO: Comprehensive error handling
async function fetchData(): Promise<Data> {
  try {
    const response = await fetch(url);
    
    if (!response.ok) {
      throw new NetworkError(`HTTP ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    if (error instanceof NetworkError) {
      logError('Network error', { error });
      throw error;
    }
    throw new MCPError('Fetch failed', { cause: error });
  }
}

// ❌ DON'T: Silent failures
async function fetchData() {
  try {
    const response = await fetch(url);
    return response.json();
  } catch (e) {
    return null; // ❌ Silent failure!
  }
}
```

## 🔄 Making Changes

### Branch Naming
```bash
# Feature branches
git checkout -b feat/add-export-functionality

# Bug fixes
git checkout -b fix/resolve-mobile-keyboard-issue

# Documentation
git checkout -b docs/update-api-documentation

# Refactoring
git checkout -b refactor/simplify-command-builder
```

### Commit Messages
Follow [Conventional Commits](https://www.conventionalcommits.org/):

```bash
# Format
<type>(<scope>): <description>

# Examples
feat(ui): add command history panel
fix(api): handle network timeout errors
docs(readme): update installation instructions
refactor(hooks): simplify useMCPCommand logic
test(api): add integration tests for execute endpoint
chore(deps): upgrade Next.js to 14.0.4
ci(workflow): add Node version check
perf(build): optimize bundle size with code splitting
```

### Code Review Checklist
Before submitting, ensure:

- [ ] TypeScript compiles without errors (`npm run type-check`)
- [ ] Linting passes (`npm run lint`)
- [ ] Tests pass (`npm test`)
- [ ] Build succeeds (`npm run build`)
- [ ] Code is formatted (`npm run format`)
- [ ] No console.logs in production code
- [ ] Accessibility tested (keyboard navigation)
- [ ] Mobile tested (iPhone Safari if possible)
- [ ] Error handling implemented
- [ ] Comments added for complex logic
- [ ] Documentation updated if needed

## 🧪 Testing

### Running Tests
```bash
# Run all tests
npm test

# Run tests in watch mode
npm test -- --watch

# Run tests with coverage
npm test -- --coverage

# Run specific test file
npm test -- CommandButton.test.tsx
```

### Writing Tests
```typescript
// __tests__/CommandButton.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { CommandButton } from '../CommandButton';

describe('CommandButton', () => {
  it('should execute command when clicked', async () => {
    const mockExecute = jest.fn().mockResolvedValue(undefined);
    
    render(<CommandButton onExecute={mockExecute} />);
    
    const button = screen.getByRole('button', { name: /execute/i });
    fireEvent.click(button);
    
    await waitFor(() => {
      expect(mockExecute).toHaveBeenCalledTimes(1);
    });
  });

  it('should show loading state', () => {
    const mockExecute = jest.fn(
      () => new Promise(resolve => setTimeout(resolve, 100))
    );
    
    render(<CommandButton onExecute={mockExecute} />);
    
    const button = screen.getByRole('button');
    fireEvent.click(button);
    
    expect(button).toHaveTextContent('Executing...');
    expect(button).toBeDisabled();
  });

  it('should handle errors gracefully', async () => {
    const mockExecute = jest.fn().mockRejectedValue(new Error('Failed'));
    const consoleSpy = jest.spyOn(console, 'error').mockImplementation();
    
    render(<CommandButton onExecute={mockExecute} />);
    
    const button = screen.getByRole('button');
    fireEvent.click(button);
    
    await waitFor(() => {
      expect(consoleSpy).toHaveBeenCalled();
    });
    
    consoleSpy.mockRestore();
  });
});
```

## 📤 Submitting Changes

### 1. Update Your Branch
```bash
# Fetch latest changes from upstream
git fetch upstream

# Rebase your branch on latest main
git rebase upstream/main

# Resolve conflicts if any
```

### 2. Push Changes
```bash
git push origin feat/your-feature-name
```

### 3. Create Pull Request

#### PR Title Format
```
<type>(<scope>): <description>

Examples:
feat(ui): add command export functionality
fix(mobile): resolve keyboard overlay issue
docs: update contributing guidelines
```

#### PR Description Template
```markdown
## Description
Brief description of changes made.

## Type of Change
- [ ] Bug fix (non-breaking change fixing an issue)
- [ ] New feature (non-breaking change adding functionality)
- [ ] Breaking change (fix or feature causing existing functionality to break)
- [ ] Documentation update
- [ ] Refactoring (no functional changes)
- [ ] Performance improvement

## Testing
Describe how you tested your changes.

## Screenshots (if applicable)
Add screenshots for UI changes.

## Checklist
- [ ] My code follows the project's code style
- [ ] I have performed a self-review
- [ ] I have commented complex code
- [ ] I have updated documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix/feature works
- [ ] New and existing tests pass locally
- [ ] Any dependent changes have been merged

## Related Issues
Closes #123
```

### 4. Code Review Process
1. Automated checks run (GitHub Actions)
2. Maintainer reviews code
3. Address feedback if requested
4. Approval and merge

## 🤖 AI-Assisted Development

### Using AI Tools (Claude, Copilot, Cursor)

This project has comprehensive AI coding instructions. Ensure your AI assistant has access to:

- `.cursorrules` - Main coding instructions
- `.github/copilot-instructions.md` - GitHub Copilot specific rules
- `CONTRIBUTING.md` - This file

#### Best Practices for AI-Assisted Development

**✅ DO:**
- Provide context about the specific feature/fix
- Ask AI to follow project coding standards
- Review AI-generated code carefully
- Test AI suggestions thoroughly
- Request explanations for complex code
- Use AI for boilerplate and repetitive tasks

**❌ DON'T:**
- Blindly accept AI suggestions
- Skip testing AI-generated code
- Ignore TypeScript errors
- Let AI write without proper types
- Commit untested AI code
- Bypass security checks

#### Example AI Prompts

**Good Prompt:**
```
Create a React component for displaying MCP command output. 
Requirements:
- TypeScript with explicit types
- Follow our project's CommandOutput interface
- Include error handling
- Use Tailwind CSS for styling
- Make it responsive (mobile-first)
- Add ARIA labels for accessibility
- Include loading state
- Add unit tests

The component should display JSON output with syntax highlighting 
and support copying to clipboard.
```

**Bad Prompt:**
```
Make a component that shows output
```

### AI Code Review Checklist

When using AI-generated code, verify:

- [ ] TypeScript types are explicit and correct
- [ ] Error handling is comprehensive
- [ ] Code follows project structure
- [ ] Tailwind CSS used (no inline styles)
- [ ] Accessibility attributes present
- [ ] Mobile-responsive design
- [ ] Security considerations addressed
- [ ] Tests included
- [ ] Documentation/comments added
- [ ] No hardcoded values or secrets

## 📚 Project Structure

```
mcp-inspector/
├── .github/
│   ├── workflows/              # GitHub Actions CI/CD
│   │   ├── deploy.yml
│   │   ├── test.yml
│   │   └── node-version-check.yml
│   ├── copilot-instructions.md # GitHub Copilot rules
│   └── dependabot.yml          # Dependency updates
├── public/                     # Static assets
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── api/               # API routes
│   │   ├── layout.tsx         # Root layout
│   │   ├── page.tsx           # Home page
│   │   └── globals.css        # Global styles
│   ├── components/            # React components
│   │   ├── ui/               # Reusable UI components
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   └── Card.tsx
│   │   ├── features/         # Feature components
│   │   │   ├── CommandBuilder/
│   │   │   ├── OutputDisplay/
│   │   │   └── ConfigManager/
│   │   └── MCPInspector.tsx  # Main component
│   ├── lib/                   # Utilities
│   │   ├── mcp/              # MCP client
│   │   ├── hooks/            # Custom hooks
│   │   └── utils/            # Helper functions
│   ├── types/                # TypeScript types
│   │   ├── mcp.ts
│   │   ├── api.ts
│   │   └── global.d.ts
│   └── config/               # Configuration
│       ├── constants.ts
│       └── env.ts
├── __tests__/                # Test files
├── .cursorrules              # Cursor AI instructions
├── .nvmrc                    # Node version
├── package.json              # Dependencies
├── tsconfig.json             # TypeScript config
├── tailwind.config.ts        # Tailwind config
├── next.config.js            # Next.js config
├── vercel.json               # Vercel config
└── README.md                 # Project documentation
```

## 🐛 Reporting Bugs

### Before Reporting
- Check existing issues
- Verify it's reproducible
- Test on latest version
- Gather relevant information

### Bug Report Template
```markdown
**Describe the bug**
Clear and concise description.

**To Reproduce**
Steps to reproduce:
1. Go to '...'
2. Click on '...'
3. See error

**Expected behavior**
What you expected to happen.

**Screenshots**
If applicable, add screenshots.

**Environment:**
- OS: [e.g. iOS, Windows]
- Browser: [e.g. Chrome, Safari]
- Version: [e.g. 1.0.0]
- Node version: [e.g. 22.7.5]

**Additional context**
Any other relevant information.
```

## 💡 Feature Requests

### Feature Request Template
```markdown
**Is your feature request related to a problem?**
Clear description of the problem.

**Describe the solution you'd like**
Clear description of what you want to happen.

**Describe alternatives you've considered**
Alternative solutions or features considered.

**Additional context**
Any other context or screenshots.

**Would you like to implement this feature?**
Yes/No - If yes, we can provide guidance.
```

## 📖 Documentation

### When to Update Documentation

Update documentation when:
- Adding new features
- Changing existing functionality
- Fixing bugs that affect usage
- Updating dependencies
- Changing configuration

### Documentation Locations
- **README.md**: Project overview, quick start
- **CONTRIBUTING.md**: This file
- **API documentation**: In code via JSDoc
- **GitHub Wiki**: Detailed guides (if applicable)

### JSDoc Comments
```typescript
/**
 * Executes an MCP command on the specified server.
 * 
 * @param config - The MCP server configuration
 * @param options - Optional execution parameters
 * @returns Promise resolving to the command response
 * @throws {ValidationError} If configuration is invalid
 * @throws {NetworkError} If connection fails
 * 
 * @example
 * ```typescript
 * const result = await executeMCPCommand({
 *   serverUrl: 'https://example.com',
 *   method: 'tools/list',
 *   transport: 'http'
 * });
 * ```
 */
export async function executeMCPCommand(
  config: MCPServerConfig,
  options?: ExecutionOptions
): Promise<MCPResponse> {
  // Implementation
}
```

## 🔒 Security

### Reporting Security Issues

**DO NOT** open public issues for security vulnerabilities.

Instead:
1. Email security concerns to [security@example.com]
2. Include detailed description
3. Provide steps to reproduce
4. Suggest a fix if possible

### Security Guidelines

- Never commit secrets or API keys
- Use environment variables for sensitive data
- Validate all user inputs
- Sanitize HTML output
- Keep dependencies updated
- Follow OWASP security practices

## 🎯 Development Tips

### Hot Tips for Productivity

1. **Use TypeScript Strict Mode**: Catch errors early
2. **Write Tests First**: TDD approach prevents bugs
3. **Use ESLint Auto-fix**: `npm run lint -- --fix`
4. **Watch Mode**: Keep tests running while coding
5. **Browser DevTools**: React DevTools, Network tab
6. **Git Hooks**: Pre-commit hooks run linting/tests

### Debugging

```typescript
// Use debugger statements (remove before commit)
debugger;

// Console with context
console.log('[CommandBuilder]', { config, state });

// React DevTools
// Install React DevTools browser extension

// Network debugging
// Use browser Network tab to inspect API calls

// Performance profiling
// Use React Profiler to find slow renders
```

### Performance Optimization

```typescript
// Use React.memo for expensive components
export const ExpensiveComponent = React.memo(({ data }) => {
  // Component logic
});

// Use useMemo for expensive computations
const processedData = useMemo(() => {
  return data.map(item => expensiveOperation(item));
}, [data]);

// Use useCallback for event handlers
const handleClick = useCallback(() => {
  // Handler logic
}, [dependencies]);

// Dynamic imports for code splitting
const HeavyComponent = dynamic(() => import('./HeavyComponent'));
```

## 🤝 Community

### Communication Channels

- **GitHub Issues**: Bug reports, feature requests
- **GitHub Discussions**: Questions, ideas, show & tell
- **Pull Requests**: Code contributions
- **Email**: For private matters

### Code of Conduct

We follow the [Contributor Covenant](https://www.contributor-covenant.org/):

- Be respectful and inclusive
- Welcome newcomers
- Focus on constructive criticism
- Accept responsibility for mistakes
- Prioritize community well-being

## 🎓 Learning Resources

### Project Technologies

- [Next.js Documentation](https://nextjs.org/docs)
- [TypeScript Handbook](https://www.typescriptlang.org/docs)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [React Testing Library](https://testing-library.com/react)
- [Model Context Protocol](https://modelcontextprotocol.io)

### DevOps & CI/CD

- [GitHub Actions](https://docs.github.com/actions)
- [Vercel Documentation](https://vercel.com/docs)
- [Docker Documentation](https://docs.docker.com)

### Best Practices

- [Clean Code JavaScript](https://github.com/ryanmcdermott/clean-code-javascript)
- [React Best Practices](https://react.dev/learn)
- [TypeScript Do's and Don'ts](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)

## 📝 Checklist Before Submitting

### Pre-Submission Checklist

- [ ] Code follows style guidelines
- [ ] TypeScript compiles without errors
- [ ] Linting passes without errors
- [ ] All tests pass
- [ ] Build succeeds
- [ ] New tests added for new features
- [ ] Documentation updated
- [ ] Commit messages follow convention
- [ ] Branch is up to date with main
- [ ] No merge conflicts
- [ ] Tested on mobile (if UI changes)
- [ ] Accessibility checked
- [ ] Security considerations addressed
- [ ] No console.logs in production code
- [ ] Environment variables documented

### Running All Checks

```bash
# Quick check before commit
npm run pre-commit

# Or run individually:
npm run type-check    # TypeScript validation
npm run lint          # ESLint
npm test              # Jest tests
npm run build         # Production build
```

## 🎉 Recognition

Contributors will be recognized in:
- README.md contributors section
- Release notes
- GitHub contributors graph

Significant contributions may also receive:
- Maintainer status
- Commit access
- Decision-making input

## 📞 Getting Help

### Stuck? Need Help?

1. **Check Documentation**: README, Wiki, JSDoc comments
2. **Search Issues**: Someone may have had the same problem
3. **GitHub Discussions**: Ask questions
4. **Discord/Slack**: Real-time help (if available)
5. **Email**: For private questions

### Good Questions Include:

- What you're trying to accomplish
- What you've tried so far
- Error messages or screenshots
- Relevant code snippets
- Environment details

## 🙏 Thank You

Thank you for contributing to MCP Inspector Pro! Your efforts help make this tool better for the entire DevOps community.

Every contribution, no matter how small, is valued:
- Code contributions
- Bug reports
- Feature suggestions
- Documentation improvements
- Community support

**Happy coding! 🚀**

---

## Quick Links

- [Project README](README.md)
- [Code of Conduct](CODE_OF_CONDUCT.md)
- [License](LICENSE)
- [Security Policy](SECURITY.md)
- [GitHub Issues](https://github.com/username/mcp-inspector/issues)
- [GitHub Discussions](https://github.com/username/mcp-inspector/discussions)
