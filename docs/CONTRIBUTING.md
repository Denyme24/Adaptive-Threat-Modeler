# Contributing to Adaptive Threat Modeler

Thank you for your interest in contributing to the Adaptive Threat Modeler! This guide will help you get started with contributing to our open-source security analysis platform.

## 🤝 Code of Conduct

We are committed to providing a welcoming and inspiring community for all. Please read and follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## 🚀 Getting Started

### Prerequisites

Before you begin, ensure you have the following installed:

- **Go** 1.23+ (for backend development)
- **Node.js** 18+ and npm (for frontend development)
- **Python** 3.11+ (for MCP agent development)
- **Git** (for version control)
- **Docker** (optional, for containerized development)

### Development Environment Setup

1. **Fork the repository** on GitHub
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/YOUR_USERNAME/Adaptive-Threat-Modeler.git
   cd Adaptive-Threat-Modeler
   ```

3. **Add the original repository as upstream**:
   ```bash
   git remote add upstream https://github.com/Denyme24/Adaptive-Threat-Modeler.git
   ```

4. **Set up each component**:

   #### Backend Setup
   ```bash
   cd backend
   go mod tidy
   cp env.example .env
   go run main.go
   ```

   #### Frontend Setup
   ```bash
   cd frontend
   npm install
   npm run dev
   ```

   #### MCP Agent Setup
   ```bash
   cd mcp
   pip install -r requirements.txt
   cp .env.example .env
   python api.py
   ```

## 📋 How to Contribute

### 1. Choose What to Work On

- Browse our [GitHub Issues](https://github.com/Denyme24/Adaptive-Threat-Modeler/issues)
- Look for issues labeled `good first issue` for beginners
- Check issues labeled `help wanted` for areas needing assistance
- Review our [Project Roadmap](ROADMAP.md) for upcoming features

### 2. Create a Feature Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b bugfix/issue-number-description
```

### 3. Make Your Changes

Follow the guidelines below for each component:

#### Backend Development (Go)

**Code Style:**
- Follow Go conventions and use `gofmt`
- Use meaningful variable and function names
- Add comments for complex logic
- Follow the existing project structure

**Testing:**
```bash
# Run tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Run specific test
go test -run TestFunctionName ./internal/services/
```

**Example: Adding a New Security Rule**

```go
// In backend/internal/services/rules.go
SecurityRule{
    ID:          "new-rule-id",
    Title:       "New Security Rule",
    Description: "Description of the security issue",
    Severity:    "high",
    Category:    "injection",
    CWE:         "CWE-XXX",
    OWASP:       "A01:2021",
    Language:    "javascript",
    Pattern: RulePattern{
        Type: "regex",
        Patterns: []string{
            `dangerous_pattern_here`,
        },
    },
    Impact:      "Description of potential impact",
    Remediation: []string{
        "Step 1 to fix",
        "Step 2 to fix",
    },
}
```

#### Frontend Development (React/TypeScript)

**Code Style:**
- Use TypeScript for all new components
- Follow React best practices and hooks
- Use existing UI components from ShadCN
- Maintain consistent styling with Tailwind CSS

**Component Structure:**
```typescript
// components/NewComponent.tsx
import { useState } from 'react';
import { Button } from '@/components/ui/button';

interface NewComponentProps {
  // Define props with types
}

export const NewComponent = ({ prop1, prop2 }: NewComponentProps) => {
  const [state, setState] = useState<string>('');

  return (
    <div className="component-container">
      {/* Component JSX */}
    </div>
  );
};
```

**Testing:**
```bash
# Run tests
npm test

# Run tests in watch mode
npm run test:watch

# Run linting
npm run lint
```

#### MCP Agent Development (Python)

**Code Style:**
- Follow PEP 8 style guidelines
- Use type hints for function parameters and return values
- Add docstrings for all functions and classes
- Use meaningful variable names

**Function Example:**
```python
from typing import List, Dict, Optional

async def analyze_security_findings(
    findings: List[Dict],
    severity_threshold: str = "medium"
) -> Optional[Dict]:
    """
    Analyze security findings and generate recommendations.
    
    Args:
        findings: List of security findings to analyze
        severity_threshold: Minimum severity level to process
        
    Returns:
        Dictionary containing analysis results or None if no findings
    """
    # Implementation here
    pass
```

**Testing:**
```bash
# Run tests
python -m pytest

# Run tests with coverage
python -m pytest --cov=.

# Run specific test
python -m pytest tests/test_api.py::test_function_name
```

### 4. Commit Your Changes

Follow conventional commit format:

```bash
git add .
git commit -m "feat: add new security rule for XSS detection"
# or
git commit -m "fix: resolve issue with GitHub API authentication"
# or
git commit -m "docs: update API documentation with new endpoints"
```

**Commit Types:**
- `feat:` New features
- `fix:` Bug fixes
- `docs:` Documentation changes
- `style:` Code style changes (formatting, etc.)
- `refactor:` Code refactoring
- `test:` Adding or updating tests
- `chore:` Maintenance tasks

### 5. Push and Create Pull Request

```bash
git push origin feature/your-feature-name
```

Then create a Pull Request on GitHub with:
- Clear title and description
- Reference to related issues
- Screenshots for UI changes
- Test results

## 🧪 Testing Guidelines

### Backend Testing

Create test files alongside your code:

```go
// internal/services/analyzer_test.go
package services

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestAnalyzer_DetectVulnerabilities(t *testing.T) {
    analyzer := NewAnalyzer()
    
    testCases := []struct {
        name     string
        input    string
        expected int
    }{
        {
            name:     "Should detect SQL injection",
            input:    `query = "SELECT * FROM users WHERE id = " + userId`,
            expected: 1,
        },
    }
    
    for _, tc := range testCases {
        t.Run(tc.name, func(t *testing.T) {
            result := analyzer.AnalyzeCode(tc.input)
            assert.Equal(t, tc.expected, len(result.Vulnerabilities))
        })
    }
}
```

### Frontend Testing

Create component tests:

```typescript
// components/__tests__/AnalysisResults.test.tsx
import { render, screen } from '@testing-library/react';
import { AnalysisResults } from '../AnalysisResults';

describe('AnalysisResults', () => {
  it('renders vulnerability count correctly', () => {
    const mockData = {
      total_vulnerabilities: 5,
      severity_breakdown: { critical: 1, high: 2, medium: 2 }
    };
    
    render(<AnalysisResults data={mockData} />);
    
    expect(screen.getByText('5 vulnerabilities found')).toBeInTheDocument();
  });
});
```

### MCP Agent Testing

Create unit tests for Python functions:

```python
# tests/test_api.py
import pytest
from unittest.mock import patch
from api import GitHubIssueCreator

@pytest.fixture
def github_creator():
    return GitHubIssueCreator()

def test_create_github_issue(github_creator):
    """Test GitHub issue creation functionality."""
    with patch('requests.post') as mock_post:
        mock_post.return_value.status_code = 201
        mock_post.return_value.json.return_value = {'html_url': 'http://example.com'}
        
        result = github_creator.create_github_issue(
            title="Test Issue",
            body="Test body"
        )
        
        assert result == 'http://example.com'
        mock_post.assert_called_once()
```

## 📝 Documentation

### Code Documentation

- **Go**: Use godoc-style comments
- **TypeScript**: Use JSDoc comments
- **Python**: Use docstrings

### README Updates

When adding new features:
1. Update the main README.md
2. Update component-specific READMEs
3. Add examples to API documentation
4. Update the changelog

### API Documentation

For new API endpoints, add examples to `docs/API_EXAMPLES.md`:

```markdown
### New Endpoint

#### Request
```bash
curl -X POST http://localhost:8080/api/v1/new-endpoint \
  -H "Content-Type: application/json" \
  -d '{"param": "value"}'
```

#### Response
```json
{
  "result": "success"
}
```

## 🎨 UI/UX Guidelines

### Design Principles

- **Consistency**: Use existing UI components and patterns
- **Accessibility**: Ensure components are accessible (ARIA labels, keyboard navigation)
- **Performance**: Optimize for fast loading and smooth interactions
- **Responsive**: Design works on all screen sizes

### Component Guidelines

- Use ShadCN UI components as base
- Follow the existing color scheme and typography
- Maintain the glassmorphism design aesthetic
- Include loading states and error handling

### Animation Guidelines

- Use Framer Motion for animations
- Keep animations subtle and purposeful
- Ensure animations don't interfere with accessibility
- Test performance on lower-end devices

## 🔍 Security Considerations

### When Contributing Security Rules

1. **Research thoroughly**: Understand the vulnerability pattern
2. **Test extensively**: Ensure low false positive rates
3. **Document clearly**: Provide clear descriptions and examples
4. **Reference standards**: Include CWE/OWASP mappings
5. **Consider impact**: Assess the severity accurately

### Security Review Process

All security-related changes undergo additional review:
- Security team review for new rules
- Penetration testing for significant changes
- Code review by maintainers
- Automated security scanning

## 🐛 Bug Reports

### Creating Good Bug Reports

Include the following information:

1. **Environment**: OS, browser, Go/Node/Python versions
2. **Steps to reproduce**: Clear, numbered steps
3. **Expected behavior**: What should happen
4. **Actual behavior**: What actually happens
5. **Screenshots**: If applicable
6. **Logs**: Relevant error messages or logs

### Bug Report Template

```markdown
## Bug Description
Brief description of the bug

## Environment
- OS: [e.g., Ubuntu 20.04]
- Browser: [e.g., Chrome 91]
- Go version: [e.g., 1.23.0]
- Node version: [e.g., 18.16.0]

## Steps to Reproduce
1. Go to '...'
2. Click on '....'
3. Scroll down to '....'
4. See error

## Expected Behavior
A clear description of what you expected to happen.

## Actual Behavior
A clear description of what actually happened.

## Screenshots
If applicable, add screenshots to help explain your problem.

## Additional Context
Add any other context about the problem here.
```

## 🎯 Feature Requests

### Creating Feature Requests

1. **Check existing issues** to avoid duplicates
2. **Describe the problem** the feature would solve
3. **Propose a solution** with implementation details
4. **Consider alternatives** and explain why your solution is best
5. **Add mockups** for UI features

## 🏆 Recognition

Contributors are recognized in several ways:

- **Contributors list** in README
- **Release notes** mention significant contributions
- **Hall of Fame** for outstanding contributors
- **Swag and stickers** for active contributors

## 📚 Resources

### Learning Resources

- [Go Documentation](https://golang.org/doc/)
- [React Documentation](https://reactjs.org/docs/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Python Documentation](https://docs.python.org/3/)
- [OWASP Security Guidelines](https://owasp.org/)

### Project Resources

- [Architecture Overview](docs/ARCHITECTURE.md)
- [API Documentation](docs/API_EXAMPLES.md)
- [Security Rules Reference](docs/SECURITY_RULES.md)
- [Deployment Guide](docs/DEPLOYMENT.md)

## 💬 Community

### Communication Channels

- **GitHub Issues**: Bug reports and feature requests
- **GitHub Discussions**: General questions and ideas
- **Discord**: Real-time chat with the community
- **Email**: security@adaptive-threat-modeler.com for security issues

### Community Guidelines

- Be respectful and inclusive
- Help others learn and grow
- Share knowledge and experiences
- Follow the code of conduct
- Focus on constructive feedback

## 🎉 Thank You

Thank you for contributing to Adaptive Threat Modeler! Your contributions help make the software security community stronger and more secure. 

Whether you're fixing a bug, adding a feature, improving documentation, or helping other users, your efforts are appreciated and make a real difference.

---

**Questions?** Feel free to reach out to the maintainers or open a discussion on GitHub. We're here to help! 🚀