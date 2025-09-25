# Sales Research Platform - Agent Guidelines

## Project Overview
A serverless sales research platform built for AWS Free Tier that enables sales teams to generate comprehensive company intelligence reports. The system scrapes multiple data sources (job boards, GitHub, company websites, news) and uses Claude AI to generate actionable insights.

## Technology Stack & Architecture

### Primary Technologies
- **Frontend**: React 18+ with TypeScript, hosted on S3 + CloudFront
- **Backend**: Python 3.11+ Lambda functions
- **Database**: DynamoDB (single-table design)
- **Authentication**: AWS Cognito
- **Orchestration**: Step Functions + SQS
- **AI**: Anthropic Claude API
- **Infrastructure**: AWS CDK (Python)

### AWS Services Used
- **Compute**: Lambda (2 types - light scraper 128MB, heavy scraper 1024MB)
- **Storage**: DynamoDB, S3
- **API**: API Gateway REST API
- **Auth**: Cognito User Pools
- **Orchestration**: Step Functions, SQS, EventBridge
- **Monitoring**: CloudWatch, X-Ray
- **Security**: IAM, Secrets Manager
- **CDN**: CloudFront

## Development Commands

### Infrastructure
```bash
# Deploy infrastructure
npm run deploy

# Deploy specific stage
npm run deploy:dev
npm run deploy:prod

# Destroy infrastructure (use carefully)
npm run destroy

# View stack outputs
npm run outputs
```

### Frontend Development
```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Type checking
npm run type-check

# Lint code
npm run lint

# Fix linting issues
npm run lint:fix

# Run tests
npm run test
```

### Backend Development
```bash
# Install dependencies (backend)
cd backend && pip install -r requirements.txt

# Install dev dependencies
pip install -r requirements-dev.txt

# Run tests
python -m pytest

# Run tests with coverage
python -m pytest --cov=src --cov-report=html

# Test specific Lambda function locally
python -m pytest tests/test_scraper_light.py

# Run type checking
python -m mypy src/

# Format code
python -m black src/ tests/

# Lint code
python -m ruff check src/ tests/

# Deploy single function
npm run deploy:function -- scraper-heavy
```

### Database Operations
```bash
# Run DynamoDB local
npm run db:local

# Seed local database
npm run db:seed

# Create table locally
npm run db:create-table

# Clear local data
npm run db:clear
```

## Code Style & Conventions

### Python Preferences
- Use Python 3.11+ features
- Follow PEP 8 style guidelines
- Use type hints for all functions and variables
- Use dataclasses or Pydantic for data models
- Enable strict mypy checking

### Naming Conventions
- **Files**: snake_case (e.g., `job_scraper.py`, `github_analyzer.py`)
- **Functions**: snake_case (e.g., `extract_job_data`, `analyze_repository`)
- **Constants**: SCREAMING_SNAKE_CASE (e.g., `MAX_CONCURRENT_SCRAPERS`)
- **Classes**: PascalCase (e.g., `JobPosting`, `CompanyData`)
- **Variables**: snake_case (e.g., `scraping_result`, `analysis_report`)

### Lambda Function Structure
```python
# Preferred Lambda handler pattern
import json
import logging
from typing import Dict, Any
from aws_lambda_powertools import Logger, Tracer, Metrics
from aws_lambda_powertools.logging import correlation_paths
from aws_lambda_powertools.metrics import MetricUnit

from src.utils.validation import validate_input
from src.utils.response import create_response

logger = Logger()
tracer = Tracer()
metrics = Metrics()

@logger.inject_lambda_context(correlation_id_path=correlation_paths.API_GATEWAY_REST)
@tracer.capture_lambda_handler
@metrics.log_metrics(capture_cold_start_metric=True)
def handler(event: Dict[str, Any], context: Any) -> Dict[str, Any]:
    """Lambda handler for processing requests."""
    try:
        # 1. Validate input
        input_data = validate_input(event)
        
        # 2. Process business logic
        result = process_request(input_data)
        
        # 3. Log metrics
        metrics.add_metric(name="RequestProcessed", unit=MetricUnit.Count, value=1)
        
        # 4. Return standardized response
        return create_response(200, result)
    
    except ValueError as e:
        logger.error(f"Validation error: {str(e)}")
        return create_response(400, {"error": str(e)})
    
    except Exception as e:
        logger.error(f"Unexpected error: {str(e)}", extra={"error": str(e)})
        return create_response(500, {"error": "Internal server error"})
```

### DynamoDB Patterns
- Use single-table design with composite keys
- Primary key format: `PK: "ORG#<company>"`, `SK: "TYPE#<entity_id>"`
- Always include `Type` attribute for filtering
- Use GSIs sparingly (max 2)
- Include `CreatedAt` and `UpdatedAt` timestamps
- Use TTL for temporary data

### Error Handling
- Always catch and log errors appropriately
- Use structured logging with Lambda context
- Return user-friendly error messages
- Never expose internal errors to frontend

## Project Structure

```
sales-researcher/
├── README.md
├── AGENTS.md
├── plan.md
├── frontend/                 # React frontend
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── vite.config.ts
├── backend/                  # Lambda functions
│   ├── src/
│   │   ├── handlers/        # Lambda entry points
│   │   ├── services/        # Business logic
│   │   ├── utils/          # Shared utilities
│   │   └── models/         # Pydantic data models
│   ├── tests/              # Test files
│   ├── requirements.txt    # Production dependencies
│   ├── requirements-dev.txt # Development dependencies
│   └── pyproject.toml      # Python project configuration
├── infrastructure/          # AWS CDK
│   ├── lib/
│   ├── bin/
│   └── package.json
└── docs/                   # Documentation
```

## Security Guidelines

### IAM Principles
- Always use least privilege principles
- Create separate IAM roles per Lambda function
- Never use wildcard permissions in production
- Use resource-specific permissions where possible

### Secrets Management
- Store all API keys in AWS Secrets Manager
- Never commit secrets to repository
- Use environment variables for non-sensitive config
- Rotate secrets regularly

### Data Privacy
- Implement TTL on personal data in DynamoDB
- Anonymize data for analytics
- Provide data deletion capabilities (GDPR compliance)
- Log access to sensitive data

## Cost Optimization

### AWS Free Tier Limits (Monitor These)
- Lambda: 400K GB-seconds/month
- API Gateway: 1M requests/month
- DynamoDB: 25GB storage + 25 RCU/WCU
- S3: 5GB storage + 20K GET/2K PUT requests
- Cognito: 50K MAU

### Cost Control Measures
- Set reserved concurrency on expensive Lambda functions
- Use DynamoDB on-demand pricing with alerts
- Implement S3 lifecycle policies
- Cache Claude API responses to reduce costs
- Monitor daily spend with CloudWatch alarms

### Performance Optimization
- Keep Lambda functions warm with CloudWatch Events
- Use Lambda layers for shared dependencies
- Optimize memory allocation with AWS Lambda Power Tuning
- Enable X-Ray tracing for performance insights

## Testing Strategy

### Unit Tests
- Test Lambda handlers in isolation
- Mock AWS SDK calls using moto
- Use pytest with pytest-mock
- Aim for >80% code coverage
- Test data models with Pydantic validation

### Integration Tests
- Test against DynamoDB Local
- Use AWS SAM for local API testing
- Test Step Functions workflows locally
- Use pytest-asyncio for async operations

### End-to-End Tests
- Use Playwright for frontend testing
- Test critical user journeys
- Run against staging environment

### Testing Requirements by Phase
Each development phase MUST include comprehensive tests:

#### Phase 1 Testing Requirements
```bash
# Must pass before proceeding to Phase 2
python -m pytest tests/test_handlers/ -v
python -m pytest tests/test_models/ -v
python -m pytest tests/test_utils/ -v
python -m mypy src/
python -m black --check src/ tests/
python -m ruff check src/ tests/
```

#### Phase 2 Testing Requirements
```bash
# Must pass before proceeding to Phase 3
python -m pytest tests/test_scrapers/ -v
python -m pytest tests/test_integrations/ -v
python -m pytest --cov=src --cov-report=term-missing
# Coverage must be >80%
```

#### Phase 3 Testing Requirements
```bash
# Must pass before proceeding to Phase 4
python -m pytest tests/test_ai_processing/ -v
python -m pytest tests/test_analytics/ -v
python -m pytest tests/integration/ -v
```

#### Phase 4 Testing Requirements
```bash
# Must pass before production deployment
python -m pytest --cov=src --cov-report=html
npm run test:e2e
python -m pytest tests/load/ -v
# Performance benchmarks must pass
```

## Deployment & CI/CD

### Environments
- **dev**: Development environment for testing
- **staging**: Pre-production testing
- **prod**: Production environment

### GitHub Actions Workflow
- Run tests on all PRs
- Deploy to staging on merge to main
- Manual approval for production deployments
- Run security scans on dependencies

## Scraping Guidelines

### Rate Limiting
- Implement exponential backoff with jitter
- Respect robots.txt files
- Use randomized delays between requests
- Monitor for IP blocking

### Data Sources Priority
1. **Primary**: Company websites, GitHub API, job board APIs
2. **Secondary**: RSS feeds, news APIs, social media APIs
3. **Manual**: Conference speakers, community data

### Scraping Ethics
- Always check and respect robots.txt
- Implement respectful crawling delays (2-5 seconds)
- Never overwhelm target servers
- Cache results to avoid redundant requests
- Have legal review scraping targets

## Claude API Integration

### Prompt Engineering
- Cache prompts in DynamoDB to reduce costs
- Use structured prompts with clear instructions
- Implement prompt versioning for A/B testing
- Monitor token usage and costs

### Best Practices
- Always validate Claude responses
- Implement fallback for API failures
- Use appropriate model for task complexity
- Log all AI interactions for analysis

## Debugging & Monitoring

### CloudWatch Logs
- Use structured logging (JSON format)
- Include request IDs for tracing
- Log performance metrics
- Set appropriate log retention periods

### X-Ray Tracing
- Enable on all Lambda functions
- Trace external API calls
- Monitor cold start impacts
- Set up performance alerts

### Alarms & Notifications
- Lambda error rates >5%
- DynamoDB throttling events
- API Gateway 4xx/5xx rates >10%
- AWS costs approaching free tier limits

## Common Issues & Solutions

### Lambda Cold Starts
- Use provisioned concurrency for critical functions
- Optimize function size and dependencies
- Consider connection pooling for databases

### DynamoDB Throttling
- Use on-demand billing mode
- Implement exponential backoff in SDK
- Monitor consumed capacity units

### Scraping Failures
- Implement retry logic with SQS DLQ
- Use different scrapers for different site types
- Monitor success rates per target

## Environment Variables

### Required for Lambda Functions
```bash
# Common across all functions
DYNAMODB_TABLE_NAME=sales-research
REGION=us-east-1
STAGE=dev

# Scraper functions
CLAUDE_API_KEY_SECRET_ARN=arn:aws:secretsmanager:...
GITHUB_TOKEN_SECRET_ARN=arn:aws:secretsmanager:...
SCRAPING_DELAY_MS=3000
MAX_CONCURRENT_REQUESTS=5

# Frontend
VITE_API_GATEWAY_URL=https://api-id.execute-api.region.amazonaws.com/stage
VITE_COGNITO_USER_POOL_ID=us-east-1_example
VITE_COGNITO_CLIENT_ID=example-client-id
```

## Quick Start Commands

```bash
# Initialize new feature development
npm run setup:dev          # Sets up local dev environment
npm run create:lambda       # Scaffolds new Lambda function
npm run create:component    # Scaffolds new React component

# Daily development workflow
npm run dev                 # Start frontend dev server
npm run lambda:local        # Start local Lambda development
npm run test:watch         # Run tests in watch mode

# Pre-deployment checks
npm run typecheck          # Check TypeScript (frontend)
python -m mypy src/        # Check Python types (backend)
npm run lint               # Check frontend code style
python -m black src/ tests/ && python -m ruff check src/ tests/  # Format and lint Python
npm run test              # Run frontend tests
python -m pytest --cov=src --cov-report=term-missing  # Run backend tests with coverage
npm run build             # Build for production

# Deployment
npm run deploy:dev        # Deploy to development
npm run deploy:prod       # Deploy to production (requires approval)
```

---

*This file should be updated as the project evolves. When adding new commands, services, or patterns, please update this documentation accordingly.*
