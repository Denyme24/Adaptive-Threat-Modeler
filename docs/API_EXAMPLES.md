# API Examples

This document provides comprehensive examples of how to use the Adaptive Threat Modeler API.

## Base URL

All API endpoints are available at: `http://localhost:8080/api/v1`

## Authentication

Currently, the API does not require authentication. In production environments, implement proper authentication mechanisms.

## Analysis Endpoints

### 1. Analyze GitHub Repository

Analyze a public or private GitHub repository.

#### Request

```bash
curl -X POST http://localhost:8080/api/v1/analyze/github \
  -H "Content-Type: application/json" \
  -d '{
    "repo_url": "https://github.com/Denyme24/sample-vulnerable-app",
    "branch": "main"
  }'
```

#### Response

```json
{
  "analysis_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "started",
  "message": "Analysis started for repository",
  "created_at": "2024-01-15T10:30:00Z"
}
```

### 2. Analyze Uploaded File

Upload and analyze a ZIP file containing source code.

#### Request

```bash
curl -X POST http://localhost:8080/api/v1/analyze/upload \
  -F "file=@/path/to/your/code.zip"
```

#### Response

```json
{
  "analysis_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "started", 
  "message": "File uploaded and analysis started",
  "file_size": 1024000,
  "created_at": "2024-01-15T10:35:00Z"
}
```

### 3. Get Analysis Status

Check the current status of an ongoing analysis.

#### Request

```bash
curl http://localhost:8080/api/v1/analysis/550e8400-e29b-41d4-a716-446655440000/status
```

#### Response

```json
{
  "analysis_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "in_progress",
  "progress": 65,
  "current_stage": "vulnerability_detection",
  "total_stages": 8,
  "stages": [
    {
      "name": "project_detection",
      "status": "completed",
      "duration": 2.3
    },
    {
      "name": "language_detection", 
      "status": "completed",
      "duration": 1.1
    },
    {
      "name": "framework_detection",
      "status": "completed", 
      "duration": 0.8
    },
    {
      "name": "ast_parsing",
      "status": "completed",
      "duration": 15.2
    },
    {
      "name": "vulnerability_detection",
      "status": "in_progress",
      "duration": null
    }
  ],
  "estimated_completion": "2024-01-15T10:45:00Z"
}
```

### 4. Get Analysis Results

Retrieve complete analysis results once the analysis is finished.

#### Request

```bash
curl http://localhost:8080/api/v1/analysis/550e8400-e29b-41d4-a716-446655440000
```

#### Response

```json
{
  "analysis_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "created_at": "2024-01-15T10:30:00Z",
  "completed_at": "2024-01-15T10:44:30Z",
  "duration": 870.5,
  "project_info": {
    "name": "sample-vulnerable-app",
    "languages": ["javascript", "python", "dockerfile"],
    "frameworks": ["express", "flask"],
    "total_files": 156,
    "total_lines": 12450,
    "services": [
      {
        "name": "express_service",
        "type": "api",
        "endpoints": [
          {
            "path": "/api/users",
            "method": "GET",
            "authentication": false
          },
          {
            "path": "/api/users/:id",
            "method": "DELETE", 
            "authentication": false
          }
        ]
      }
    ],
    "dependencies": {
      "express": "4.18.2",
      "flask": "2.3.2",
      "mysql2": "3.6.0"
    },
    "config_files": ["package.json", "requirements.txt", "Dockerfile"]
  },
  "vulnerabilities": [
    {
      "id": "vuln_001",
      "rule_id": "javascript-sql-injection",
      "title": "SQL Injection Vulnerability",
      "description": "Raw SQL query construction with user input",
      "severity": "critical",
      "category": "injection",
      "cwe": "CWE-89",
      "owasp": "A03:2021",
      "file_path": "src/controllers/userController.js",
      "line_number": 45,
      "column_number": 12,
      "code_snippet": "const query = `SELECT * FROM users WHERE id = ${userId}`;",
      "impact": "Attackers can execute arbitrary SQL commands",
      "remediation": [
        "Use parameterized queries or prepared statements",
        "Validate and sanitize user input",
        "Implement proper SQL escaping"
      ],
      "references": [
        "https://owasp.org/www-community/attacks/SQL_Injection",
        "https://cwe.mitre.org/data/definitions/89.html"
      ]
    },
    {
      "id": "vuln_002", 
      "rule_id": "javascript-xss",
      "title": "Cross-Site Scripting (XSS)",
      "description": "Unescaped user input in DOM manipulation",
      "severity": "high",
      "category": "injection",
      "cwe": "CWE-79",
      "owasp": "A03:2021",
      "file_path": "public/js/profile.js",
      "line_number": 23,
      "column_number": 8,
      "code_snippet": "element.innerHTML = userBio;",
      "impact": "Attackers can execute malicious scripts in user browsers",
      "remediation": [
        "Use textContent instead of innerHTML for user data",
        "Implement proper HTML escaping",
        "Use Content Security Policy (CSP)"
      ]
    }
  ],
  "summary": {
    "total_vulnerabilities": 47,
    "risk_score": 8.7,
    "security_posture": "Poor",
    "severity_breakdown": {
      "critical": 3,
      "high": 12,
      "medium": 18,
      "low": 14
    },
    "category_breakdown": {
      "injection": 15,
      "authentication": 8,
      "crypto": 6,
      "authorization": 4,
      "misconfiguration": 14
    },
    "top_risks": [
      "SQL Injection in user authentication",
      "Missing authentication on admin endpoints", 
      "Hardcoded database credentials",
      "Unencrypted sensitive data transmission",
      "Missing input validation"
    ]
  },
  "threat_map": {
    "components": [
      {
        "id": "web_server",
        "name": "Express Web Server",
        "type": "service",
        "technologies": ["nodejs", "express"],
        "position": {"x": 100, "y": 100}
      },
      {
        "id": "database",
        "name": "MySQL Database", 
        "type": "datastore",
        "technologies": ["mysql"],
        "position": {"x": 300, "y": 100}
      },
      {
        "id": "user_browser",
        "name": "User Browser",
        "type": "external",
        "technologies": ["browser"],
        "position": {"x": 100, "y": 300}
      }
    ],
    "flows": [
      {
        "id": "flow_001",
        "source": "user_browser",
        "target": "web_server",
        "data_type": "HTTP Request",
        "protocols": ["HTTPS"],
        "threats": ["SQL Injection", "XSS"]
      },
      {
        "id": "flow_002", 
        "source": "web_server",
        "target": "database",
        "data_type": "SQL Query",
        "protocols": ["MySQL"],
        "threats": ["SQL Injection", "Data Exposure"]
      }
    ]
  },
  "recommendations": [
    {
      "id": "rec_001",
      "priority": "critical",
      "title": "Implement Parameterized Queries",
      "description": "Replace all string concatenation in SQL queries with parameterized queries",
      "effort": "medium",
      "impact": "high",
      "files_affected": [
        "src/controllers/userController.js",
        "src/controllers/adminController.js"
      ]
    },
    {
      "id": "rec_002",
      "priority": "high", 
      "title": "Add Authentication Middleware",
      "description": "Implement proper authentication for all admin endpoints",
      "effort": "high",
      "impact": "high",
      "files_affected": [
        "src/routes/admin.js",
        "src/middleware/auth.js"
      ]
    }
  ]
}
```

### 5. Get Analysis Logs

Retrieve detailed analysis logs with color coding and formatting.

#### Request

```bash
curl http://localhost:8080/api/v1/analysis/550e8400-e29b-41d4-a716-446655440000/logs
```

#### Response

```json
{
  "analysis_id": "550e8400-e29b-41d4-a716-446655440000",
  "logs": "=== PROJECT ANALYSIS STARTED ===\nAnalysis ID: 550e8400-e29b-41d4-a716-446655440000\nProject Path: /tmp/analysis_550e8400-e29b-41d4-a716-446655440000/repo\nDetected Languages: [javascript python dockerfile]\nDetected Frameworks: [express flask]\nServices Found: 2\nDependencies: 15\nConfig Files: [package.json requirements.txt Dockerfile]\n===============================\n\nFound 3 vulnerability(ies) in file: src/controllers/userController.js\n  - [critical] SQL Injection Vulnerability at line 45\n  - [high] Missing Input Validation at line 38\n  - [medium] Weak Error Handling at line 52\n\n...",
  "timestamp": "2024-01-15T10:44:30Z"
}
```

## Detection Endpoints

### 1. Detect Languages

Analyze a project to detect programming languages.

#### Request

```bash
curl -X POST http://localhost:8080/api/v1/detect/languages \
  -H "Content-Type: application/json" \
  -d '{
    "project_path": "/path/to/project"
  }'
```

#### Response

```json
{
  "languages": [
    {
      "name": "javascript",
      "confidence": 0.95,
      "file_count": 45,
      "line_count": 8500
    },
    {
      "name": "python", 
      "confidence": 0.88,
      "file_count": 12,
      "line_count": 2100
    },
    {
      "name": "dockerfile",
      "confidence": 1.0,
      "file_count": 2,
      "line_count": 50
    }
  ],
  "total_files": 59,
  "total_lines": 10650
}
```

### 2. Detect Frameworks

Analyze a project to detect frameworks and libraries.

#### Request

```bash
curl -X POST http://localhost:8080/api/v1/detect/frameworks \
  -H "Content-Type: application/json" \
  -d '{
    "project_path": "/path/to/project",
    "languages": ["javascript", "python"]
  }'
```

#### Response

```json
{
  "frameworks": [
    {
      "name": "express",
      "language": "javascript",
      "confidence": 0.92,
      "version": "4.18.2",
      "files": ["package.json", "src/app.js"]
    },
    {
      "name": "flask",
      "language": "python", 
      "confidence": 0.85,
      "version": "2.3.2",
      "files": ["requirements.txt", "app.py"]
    }
  ]
}
```

## Rules Endpoints

### 1. Get All Rules

Retrieve all available security rules.

#### Request

```bash
curl http://localhost:8080/api/v1/rules
```

#### Response

```json
{
  "total_rules": 127,
  "rules": [
    {
      "id": "javascript-sql-injection",
      "title": "SQL Injection Vulnerability",
      "description": "Raw SQL queries with string formatting",
      "severity": "critical",
      "category": "injection",
      "cwe": "CWE-89",
      "owasp": "A03:2021",
      "language": "javascript",
      "pattern_type": "regex"
    },
    {
      "id": "python-hardcoded-secret",
      "title": "Hardcoded Secret",
      "description": "Hardcoded passwords or API keys",
      "severity": "high", 
      "category": "crypto",
      "cwe": "CWE-798",
      "language": "python",
      "pattern_type": "regex"
    }
  ]
}
```

### 2. Get Rules for Language

Retrieve security rules for a specific programming language.

#### Request

```bash
curl http://localhost:8080/api/v1/rules/javascript
```

#### Response

```json
{
  "language": "javascript",
  "total_rules": 23,
  "rules": [
    {
      "id": "javascript-sql-injection",
      "title": "SQL Injection Vulnerability", 
      "description": "Raw SQL queries with string formatting",
      "severity": "critical",
      "category": "injection",
      "cwe": "CWE-89",
      "owasp": "A03:2021"
    },
    {
      "id": "javascript-xss",
      "title": "Cross-Site Scripting (XSS)",
      "description": "Unescaped user input in DOM manipulation", 
      "severity": "high",
      "category": "injection",
      "cwe": "CWE-79",
      "owasp": "A03:2021"
    }
  ]
}
```

## Commit Analysis Endpoints

### 1. Analyze Git Commit

Analyze a specific git commit for security changes.

#### Request

```bash
curl -X POST http://localhost:8080/api/v1/analyze/commit \
  -H "Content-Type: application/json" \
  -d '{
    "commit_hash": "a1b2c3d4e5f6",
    "repository_path": "/path/to/repo"
  }'
```

#### Response

```json
{
  "commit_hash": "a1b2c3d4e5f6",
  "analysis_id": "550e8400-e29b-41d4-a716-446655440002",
  "status": "started",
  "message": "Commit analysis started"
}
```

### 2. Get Latest Commit Analysis

Retrieve the most recent commit analysis.

#### Request

```bash
curl http://localhost:8080/api/v1/commits/latest
```

#### Response

```json
{
  "commit_hash": "a1b2c3d4e5f6",
  "commit_message": "Add user authentication system",
  "author": "john.doe@example.com",
  "timestamp": "2024-01-15T09:30:00Z",
  "files_changed": [
    {
      "file_name": "src/auth.js",
      "status": "added",
      "additions": 120,
      "deletions": 0
    },
    {
      "file_name": "src/middleware/auth.js", 
      "status": "modified",
      "additions": 25,
      "deletions": 5
    }
  ],
  "security_impact": {
    "risk_level": "medium",
    "new_vulnerabilities": 2,
    "fixed_vulnerabilities": 1,
    "net_risk_change": "+1"
  },
  "vulnerabilities": [
    {
      "id": "commit_vuln_001",
      "title": "Weak Password Validation", 
      "severity": "medium",
      "file": "src/auth.js",
      "line": 45
    }
  ]
}
```

## System Endpoints

### 1. Health Check

Check system health and status.

#### Request

```bash
curl http://localhost:8080/health
```

#### Response

```json
{
  "status": "healthy",
  "service": "adaptive-threat-modeler",
  "version": "1.0.0",
  "uptime": 3600.5,
  "timestamp": "2024-01-15T10:45:00Z"
}
```

### 2. System Information

Get detailed system information and capabilities.

#### Request

```bash
curl http://localhost:8080/api/v1/info
```

#### Response

```json
{
  "version": "1.0.0",
  "build_time": "2024-01-10T15:30:00Z",
  "git_commit": "a1b2c3d4e5f6",
  "supported_languages": [
    "go", "javascript", "typescript", "python", 
    "java", "php", "ruby", "csharp", "cpp", "hcl"
  ],
  "supported_frameworks": [
    "fiber", "gin", "echo", "express", "fastapi", 
    "django", "spring", "react", "vue", "angular"
  ],
  "analysis_capabilities": [
    "static_analysis", "ast_parsing", "pattern_matching",
    "taint_analysis", "threat_modeling", "dataflow_analysis"
  ],
  "integrations": [
    "github", "slack", "openai", "semgrep"
  ]
}
```

## Error Responses

All endpoints return appropriate HTTP status codes and error messages.

### Common Error Responses

#### 400 Bad Request

```json
{
  "error": true,
  "message": "Invalid request format",
  "details": "Missing required field: repo_url"
}
```

#### 404 Not Found

```json
{
  "error": true, 
  "message": "Analysis not found",
  "analysis_id": "invalid-id"
}
```

#### 500 Internal Server Error

```json
{
  "error": true,
  "message": "Internal server error",
  "details": "Analysis engine temporarily unavailable"
}
```

## Rate Limiting

The API implements rate limiting to ensure fair usage:

- **Rate Limit**: 100 requests per minute per IP
- **Headers**: 
  - `X-RateLimit-Limit`: Maximum requests per minute
  - `X-RateLimit-Remaining`: Remaining requests
  - `X-RateLimit-Reset`: Unix timestamp when limit resets

#### Rate Limit Exceeded Response

```json
{
  "error": true,
  "message": "Rate limit exceeded",
  "retry_after": 60
}
```

## SDK Examples

### JavaScript/Node.js

```javascript
const axios = require('axios');

class ThreatModelerAPI {
  constructor(baseURL = 'http://localhost:8080/api/v1') {
    this.baseURL = baseURL;
  }

  async analyzeRepository(repoUrl, branch = 'main') {
    const response = await axios.post(`${this.baseURL}/analyze/github`, {
      repo_url: repoUrl,
      branch: branch
    });
    return response.data;
  }

  async getAnalysisStatus(analysisId) {
    const response = await axios.get(`${this.baseURL}/analysis/${analysisId}/status`);
    return response.data;
  }

  async getAnalysisResults(analysisId) {
    const response = await axios.get(`${this.baseURL}/analysis/${analysisId}`);
    return response.data;
  }
}

// Usage
const api = new ThreatModelerAPI();
const analysis = await api.analyzeRepository('https://github.com/user/repo');
console.log(`Analysis started: ${analysis.analysis_id}`);
```

### Python

```python
import requests
import time

class ThreatModelerAPI:
    def __init__(self, base_url='http://localhost:8080/api/v1'):
        self.base_url = base_url
    
    def analyze_repository(self, repo_url, branch='main'):
        response = requests.post(f'{self.base_url}/analyze/github', json={
            'repo_url': repo_url,
            'branch': branch
        })
        return response.json()
    
    def wait_for_analysis(self, analysis_id, timeout=600):
        start_time = time.time()
        while time.time() - start_time < timeout:
            status = self.get_analysis_status(analysis_id)
            if status['status'] == 'completed':
                return self.get_analysis_results(analysis_id)
            elif status['status'] == 'failed':
                raise Exception(f"Analysis failed: {status.get('error')}")
            time.sleep(5)
        raise TimeoutError("Analysis timed out")
    
    def get_analysis_status(self, analysis_id):
        response = requests.get(f'{self.base_url}/analysis/{analysis_id}/status')
        return response.json()
    
    def get_analysis_results(self, analysis_id):
        response = requests.get(f'{self.base_url}/analysis/{analysis_id}')
        return response.json()

# Usage
api = ThreatModelerAPI()
analysis = api.analyze_repository('https://github.com/user/repo')
results = api.wait_for_analysis(analysis['analysis_id'])
print(f"Found {results['summary']['total_vulnerabilities']} vulnerabilities")
```

This comprehensive API documentation provides all the examples needed to integrate with the Adaptive Threat Modeler API effectively.