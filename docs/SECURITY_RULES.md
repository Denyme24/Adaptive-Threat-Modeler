# Security Rules Reference

## Overview

The Adaptive Threat Modeler includes a comprehensive set of built-in security rules that detect vulnerabilities across multiple programming languages and frameworks. This document provides a complete reference of all available rules.

## Rule Categories

### 1. Injection Vulnerabilities (OWASP A03:2021)

#### SQL Injection (CWE-89)
- **Languages**: Python, JavaScript, Java, PHP
- **Detection**: Raw SQL queries with string formatting
- **Examples**: 
  - `cursor.execute("SELECT * FROM users WHERE id = " + user_id)`
  - `query = f"SELECT * FROM {table} WHERE id = {id}"`

#### Cross-Site Scripting (CWE-79)
- **Languages**: JavaScript, TypeScript, HTML
- **Detection**: Unescaped user input in DOM manipulation
- **Examples**: 
  - `element.innerHTML = userInput`
  - `document.write(untrustedData)`

#### Command Injection (CWE-78)
- **Languages**: Python, JavaScript, Shell
- **Detection**: Unsafe execution of system commands
- **Examples**: 
  - `os.system(user_input)`
  - `exec(shell_command)`

### 2. Cryptographic Issues (CWE-327)

#### Weak Cryptographic Algorithms
- **Languages**: All
- **Detection**: Use of deprecated crypto algorithms
- **Examples**: 
  - MD5, SHA1 for hashing
  - DES, RC4 for encryption
  - Weak random number generators

#### Hardcoded Secrets (CWE-798)
- **Languages**: All
- **Detection**: Hardcoded passwords, API keys, tokens
- **Examples**: 
  - `password = "admin123"`
  - `api_key = "sk-1234567890abcdef"`

### 3. Infrastructure Misconfigurations

#### AWS S3 Bucket Misconfigurations
- **Languages**: HCL/Terraform
- **Detection**: Publicly accessible S3 buckets
- **Examples**: 
  - Public read/write permissions
  - Missing encryption settings

#### Database Security Issues
- **Languages**: HCL/Terraform, SQL
- **Detection**: Unencrypted databases, weak access controls
- **Examples**: 
  - Unencrypted RDS instances
  - Missing database passwords

### 4. Authentication & Authorization (OWASP A07:2021)

#### Missing Authentication
- **Languages**: JavaScript, Python, Java
- **Detection**: Unprotected API endpoints
- **Examples**: 
  - Routes without authentication middleware
  - Admin functions without access control

#### Weak Session Management
- **Languages**: All web frameworks
- **Detection**: Insecure session handling
- **Examples**: 
  - Sessions without expiration
  - Predictable session tokens

### 5. Security Logging & Monitoring (OWASP A09:2021)

#### Insufficient Logging
- **Languages**: All
- **Detection**: Missing security event logging
- **Examples**: 
  - Authentication failures not logged
  - Sensitive operations without audit trails

#### Information Disclosure
- **Languages**: All
- **Detection**: Sensitive information exposure
- **Examples**: 
  - Debug statements in production
  - Detailed error messages

## Language-Specific Rules

### Go Rules
- Unsafe pointer operations
- Race condition patterns
- Goroutine leaks
- Interface{} type assertions

### JavaScript/TypeScript Rules
- Prototype pollution
- RegExp DOS vulnerabilities
- Unsafe `eval()` usage
- Missing input validation

### Python Rules
- Pickle deserialization vulnerabilities
- Flask/Django security issues
- Unsafe file operations
- Import injection

### Java Rules
- Deserialization vulnerabilities
- XML external entity (XXE) attacks
- Spring Security misconfigurations
- Unsafe reflection usage

### HCL/Terraform Rules
- AWS resource misconfigurations
- Azure security issues
- GCP IAM problems
- Kubernetes security issues

### Shell Script Rules
- Command injection vulnerabilities
- Unsafe file operations
- Missing input validation
- Privilege escalation risks

## Framework-Specific Rules

### Web Frameworks
- **Express.js**: Missing security headers, CORS misconfigurations
- **Flask/Django**: CSRF protection, SQL injection via ORM
- **Spring Boot**: Security misconfigurations, authentication bypass
- **Fiber/Gin**: Missing middleware, unsafe route handlers

### Cloud Platforms
- **AWS**: IAM policy issues, resource exposure
- **Azure**: Storage account misconfigurations, network security
- **GCP**: Service account vulnerabilities, firewall rules

## Rule Engine Configuration

### Custom Rules

You can add custom security rules by creating rule files in the `backend/internal/services/rules.go` file:

```go
SecurityRule{
    ID:          "custom-rule-id",
    Title:       "Custom Security Rule",
    Description: "Description of the security issue",
    Severity:    "high",
    Category:    "custom",
    CWE:         "CWE-XXX",
    OWASP:       "A01:2021",
    Language:    "go",
    Pattern: RulePattern{
        Type: "regex",
        Patterns: []string{
            `dangerous_function\([^)]*\)`,
        },
    },
    Impact:      "Description of potential impact",
    Remediation: []string{
        "Step 1 to fix",
        "Step 2 to fix",
    },
}
```

### Rule Pattern Types

1. **Regex Patterns**: Regular expressions for text matching
2. **AST Patterns**: Abstract syntax tree pattern matching
3. **Dataflow Patterns**: Taint analysis patterns
4. **Semantic Patterns**: Code semantic analysis

### Severity Levels

- **Critical**: Immediate security risk, potential data breach
- **High**: Significant security vulnerability
- **Medium**: Moderate security issue
- **Low**: Minor security concern or best practice violation

## Rule Testing

### Testing Custom Rules

```bash
# Test specific rule
go test -run TestSecurityRule_CustomRule ./internal/services/

# Test all rules
go test ./internal/services/ -v
```

### Rule Validation

All rules are validated for:
- Pattern syntax correctness
- CWE/OWASP mapping accuracy
- Remediation step completeness
- False positive rate

## Contributing New Rules

### Rule Development Process

1. **Identify Security Issue**: Research the vulnerability pattern
2. **Create Test Cases**: Develop positive and negative test cases
3. **Implement Rule**: Add rule definition with patterns
4. **Test Rule**: Validate against test cases
5. **Document Rule**: Add to this reference guide
6. **Submit PR**: Follow contribution guidelines

### Rule Quality Guidelines

- **Precision**: Minimize false positives
- **Recall**: Catch all relevant vulnerabilities
- **Performance**: Ensure efficient pattern matching
- **Documentation**: Provide clear descriptions and examples

## Updates and Maintenance

### Regular Updates

- New vulnerability patterns added monthly
- CWE/OWASP mappings updated annually
- Framework-specific rules updated with new releases
- Community contributions integrated continuously

### Deprecation Policy

- Deprecated rules marked for 6 months before removal
- Migration guides provided for rule updates
- Backward compatibility maintained when possible

## Support

For questions about security rules:
- Open an issue on GitHub
- Join our Discord community
- Email: security-rules@adaptive-threat-modeler.com