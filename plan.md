# Sales Research Platform - Project Plan

## Overview
A comprehensive sales research platform that enables sales teams to input a company name and customer email to generate detailed intelligence reports covering technical infrastructure, team composition, buying signals, and tailored talking points. **Optimized for AWS Free Tier serverless deployment.**

## Serverless Architecture Overview

### System Components
```
┌─────────────────────────────┐     ┌─────────────────────────┐
│  S3 Static Site + CloudFront│────▶│    API Gateway          │
│  (React/Vue Frontend)       │     │    + Cognito Auth       │
└─────────────────────────────┘     └──────────┬──────────────┘
                                              │
                                              ▼
                                    ┌─────────────────────────┐
                                    │  Lambda: REST API       │
                                    │  (Node.js/Python)       │
                                    └──────────┬──────────────┘
                                              │ enqueues jobs
                                              ▼
                                    ┌─────────────────────────┐
                                    │      SQS Queue          │
                                    └──────────┬──────────────┘
                                              │ triggers
                                              ▼
                                    ┌─────────────────────────┐
                                    │    Step Functions       │
                                    │   (Job Orchestration)   │
                                    └─┬────┬────┬─────────────┘
                                      │    │    │
                     ┌────────────────┘    │    └─────────────────┐
                     ▼                     ▼                      ▼
          ┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐
          │ Lambda: Light       │ │ Lambda: Heavy       │ │ Lambda: AI          │
          │ Scraper (Cheerio)   │ │ Scraper (Puppeteer) │ │ Processing (Claude) │
          │ 128-256MB, 30s      │ │ 1024MB, 5min        │ │ 512MB, 2min         │
          └─────────┬───────────┘ └─────────┬───────────┘ └─────────┬───────────┘
                    │                       │                       │
                    └───────────────────────┼───────────────────────┘
                                            ▼
                                  ┌─────────────────────────┐
                                  │     DynamoDB            │
                                  │  (Single Table Design)  │
                                  │  + S3 (Raw Data Store)  │
                                  └─────────────────────────┘
```

### AWS Free Tier Service Mapping
| Component | Service | Free Tier Limits |
|-----------|---------|------------------|
| Frontend | S3 + CloudFront | 5GB storage, 20K requests/month |
| API | API Gateway + Lambda | 1M requests, 400K GB-seconds |
| Auth | Cognito | 50K MAU free |
| Database | DynamoDB | 25GB storage, on-demand billing |
| Queue | SQS | 1M requests/month |
| Orchestration | Step Functions | 4K transitions/month |
| Storage | S3 | 5GB storage, 20K GET requests |
| Monitoring | CloudWatch + X-Ray | 10 alarms, 100K traces |

## Technology Stack

### Backend (Serverless)
- **Runtime**: Node.js 18+ Lambda functions
- **Database**: DynamoDB (single-table design) + S3 for blob storage
- **Queue System**: SQS + Step Functions for orchestration
- **Web Scraping**: 
  - Light: Cheerio + axios (128-256MB Lambda)
  - Heavy: Puppeteer + chromium-aws-lambda (1024MB Lambda)
- **AI Integration**: Direct Claude API calls from Lambda
- **Authentication**: AWS Cognito with JWT

### Frontend (Static Site)
- **Framework**: React with Vite or Next.js static export
- **UI Components**: Tailwind CSS + HeadlessUI
- **State Management**: Zustand or React Context
- **Data Visualization**: Chart.js or Recharts
- **Hosting**: S3 Static Website + CloudFront CDN

### Infrastructure (AWS Free Tier)
- **IaC**: AWS CDK or Serverless Framework
- **Deployment**: GitHub Actions with AWS CLI
- **Monitoring**: CloudWatch + AWS X-Ray
- **Security**: IAM roles with least privilege

## Implementation Phases

### Phase 1: Serverless Foundation (3-4 weeks)
#### MVP Components
- [ ] AWS CDK/Serverless Framework infrastructure setup
- [ ] S3 static site hosting with React frontend
- [ ] API Gateway + basic Lambda REST API
- [ ] Cognito authentication integration
- [ ] DynamoDB single-table schema design
- [ ] SQS queue + basic Step Functions orchestration
- [ ] Simple company website scraper (Cheerio-based)
- [ ] Basic Claude API integration for text processing

#### Deliverables
- Serverless web application with basic research capabilities
- Company website content extraction via lightweight scraping
- Basic report generation stored in DynamoDB
- User authentication with Cognito
- Cost monitoring and billing alerts setup

### Phase 2: Enhanced Data Collection (4-5 weeks)
#### Serverless Data Sources Integration
- [ ] **Heavy Scraper Lambda (Puppeteer)**
  - chromium-aws-lambda layer setup
  - JavaScript-heavy site scraping
  - Reserved concurrency limits (max 5 concurrent)
  - S3 raw data storage with TTL

- [ ] **Job Boards Integration**
  - LinkedIn Jobs scraping (Puppeteer Lambda)
  - Indeed job scraping (Light Lambda)
  - AngelList API integration (Light Lambda)
  - Stack Overflow Jobs integration (Light Lambda)
  - Rate limiting and caching in S3

- [ ] **GitHub Integration**
  - GitHub API Lambda functions
  - Repository analysis with GraphQL API
  - Organization profiles and team structure
  - Technology stack detection from repo languages

- [ ] **Content Sources**
  - RSS feed parsing (Light Lambda)
  - Google News API integration
  - Company blog detection and scraping
  - HackerNews mentions via API

#### Serverless Data Processing
- [ ] DynamoDB Streams for real-time processing
- [ ] Step Functions for complex workflow orchestration
- [ ] Lambda-based sentiment analysis
- [ ] S3-based data lake for raw content storage
- [ ] CloudWatch metrics for scraping success rates

### Phase 3: Advanced Intelligence & AI Integration (3-4 weeks)
#### Serverless AI-Powered Analysis
- [ ] **Claude Integration Expansion**
  - Prompt caching in DynamoDB for cost optimization
  - Advanced text summarization and analysis
  - Technical pain point identification from scraped data
  - Buying signal detection algorithms
  - Report quality scoring and optimization

- [ ] **Intelligent Data Correlation Lambda**
  - Cross-reference job postings with GitHub activity
  - News sentiment correlation with hiring trends
  - Technology stack evolution tracking
  - Team growth velocity calculations

- [ ] **Predictive Analytics Functions**
  - Buying probability scoring algorithms
  - Contact timing optimization based on patterns
  - Budget estimation models
  - ROI calculation templates with industry benchmarks

### Phase 4: Optimization & Polish (3-4 weeks)
#### Enhanced User Experience
- [ ] Real-time job status via DynamoDB Streams + WebSocket API
- [ ] Customizable report templates stored in S3
- [ ] Bulk research via Step Functions batch processing
- [ ] Export functionality (PDF generation in Lambda)
- [ ] Advanced filtering and search in frontend

#### Performance & Cost Optimization
- [ ] Lambda power tuning for optimal memory allocation
- [ ] S3 Intelligent-Tiering for cost optimization
- [ ] CloudFront caching strategies
- [ ] DynamoDB capacity optimization
- [ ] Reserved concurrency fine-tuning
- [ ] Cost anomaly detection and alerting

## Data Collection Specifications

### Developer-Focused Data Sources

#### Job Posting Analysis
```javascript
// Data structure for job postings
{
  company: string,
  title: string,
  location: string,
  remote_policy: 'remote' | 'hybrid' | 'onsite',
  seniority_level: 'junior' | 'mid' | 'senior' | 'principal' | 'lead',
  tech_stack: string[],
  urgency_indicators: {
    posting_frequency: number,
    urgent_keywords: string[],
    salary_premium: boolean
  },
  requirements: string[],
  benefits: string[],
  team_size_indicators: number,
  posting_date: Date,
  source: string
}
```

#### GitHub Intelligence
```javascript
// Repository analysis structure
{
  organization: string,
  repositories: [{
    name: string,
    languages: { [language: string]: percentage },
    contributors: number,
    activity_score: number,
    last_updated: Date,
    stars: number,
    open_issues: number,
    code_quality_indicators: {
      has_ci_cd: boolean,
      test_coverage: number,
      documentation_quality: 'high' | 'medium' | 'low'
    }
  }],
  engineering_practices: {
    code_review_patterns: string,
    deployment_frequency: string,
    technology_adoption_speed: 'fast' | 'medium' | 'slow'
  }
}
```

### Output Report Structure

#### Executive Summary
- Company overview and key metrics
- Technology maturity assessment
- Buying signal strength (1-10 scale)
- Recommended approach strategy

#### Technical Intelligence
- Current technology stack analysis
- Engineering team structure and hiring trends
- Development workflow pain points
- Modernization opportunities

#### Buying Signals
- Recent leadership changes
- Growth indicators (team expansion, funding)
- Technical debt mentions
- Compliance/security requirements
- Developer productivity complaints

#### Tailored Talking Points
- Specific workflow improvements
- ROI calculations based on team size
- Integration complexity assessment
- Relevant case studies
- Implementation timeline suggestions

## DynamoDB Schema Design (Single Table)

### Table Structure
```javascript
// Primary Key Structure: PK + SK
{
  PK: "ORG#<company_name>",           // Partition Key
  SK: "METADATA",                     // Sort Key
  Type: "Organization",
  Name: "Company Name",
  Domain: "company.com",
  Industry: "Technology",
  CreatedAt: "2025-01-15T10:30:00Z",
  UpdatedAt: "2025-01-15T10:30:00Z",
  TTL: 1672444800                     // Optional TTL for data expiration
}

// Job Postings
{
  PK: "ORG#<company_name>",
  SK: "JOB#<job_id>",
  Type: "JobPosting",
  Title: "Senior Software Engineer",
  Location: "Remote",
  TechStack: ["Python", "AWS", "React"],
  PostedDate: "2025-01-10T00:00:00Z",
  Source: "linkedin",
  SeniorityLevel: "senior",
  UrgencyScore: 7
}

// GitHub Data
{
  PK: "ORG#<company_name>",
  SK: "GITHUB#<repo_name>",
  Type: "Repository",
  Languages: {"JavaScript": 60, "Python": 40},
  Stars: 150,
  Contributors: 8,
  LastCommit: "2025-01-14T15:22:00Z"
}
```

### Global Secondary Indexes
- GSI1: `Type` + `UpdatedAt` (for querying by entity type)
- GSI2: `Domain` + `SK` (for domain-based lookups)

## Security & Compliance (Serverless)

### AWS Security
- [ ] IAM roles with least privilege per Lambda function
- [ ] AWS Secrets Manager for API keys (Claude, GitHub, etc.)
- [ ] VPC-less Lambda functions (no VPC needed for this use case)
- [ ] S3 bucket encryption with SSE-S3 (free tier)
- [ ] API Gateway request validation and throttling
- [ ] Cognito SRP authentication (no password storage)

### Data Privacy & Compliance
- [ ] DynamoDB TTL for automatic data expiration
- [ ] GDPR compliance with data deletion capabilities
- [ ] CCPA compliance for California residents
- [ ] Audit logging with CloudTrail (free tier: 90 days)
- [ ] Data anonymization for analytics

### Web Scraping Ethics & Rate Limiting
- [ ] Respect robots.txt files in scraper logic
- [ ] Exponential backoff with jitter in Lambda functions
- [ ] SQS dead letter queues for failed scraping attempts
- [ ] Lambda reserved concurrency to prevent overwhelming targets
- [ ] S3 caching with ETags to avoid redundant scraping
- [ ] Legal review of scraping target terms of service

## Development Timeline (Serverless-Optimized)

### Month 1: Serverless Foundation
- AWS CDK infrastructure setup
- React static site with S3 + CloudFront
- Basic Lambda APIs with API Gateway
- DynamoDB schema implementation
- Cognito authentication
- Simple website scraping (Cheerio)

### Month 2: Enhanced Data Collection
- Heavy scraper Lambda with Puppeteer
- Job board integrations (LinkedIn, Indeed)
- GitHub API Lambda functions
- SQS + Step Functions orchestration
- Basic Claude integration for analysis

### Month 3: Intelligence & AI
- Advanced Claude prompt engineering
- Buying signal detection algorithms
- Data correlation and analysis functions
- Report generation optimization
- Cost monitoring and optimization

### Month 4: Polish & Production
- Real-time status updates
- Export functionality (PDF generation)
- Performance tuning and cost optimization
- Security hardening
- Production deployment and monitoring

## Resource Requirements (Serverless)

### Development Team (Smaller team due to serverless)
- 1 Full-stack developer (AWS/React specialist)
- 1 Backend developer (Node.js/Python + AWS services)
- 1 Frontend developer (React/TypeScript)

### AWS Free Tier Costs (Monthly)
- **Free Tier Services**: $0 (first 12 months for new accounts)
  - Lambda: 400K GB-seconds free
  - API Gateway: 1M requests free  
  - DynamoDB: 25GB storage free
  - S3: 5GB storage free
  - Cognito: 50K MAU free
  - CloudWatch: 10 alarms free

### Paid Services (After free tier or overages)
- Claude API usage: $50-300/month (based on usage)
- GitHub API: Free (5K requests/hour)
- Domain + Route 53: $1-2/month
- AWS overages (if any): $10-50/month
- **Total estimated monthly cost: $60-350**

### Cost Control Measures
- Reserved concurrency limits on expensive Lambda functions
- DynamoDB on-demand pricing with burst alerts
- S3 lifecycle policies for data archival
- CloudWatch billing alarms at 80% of free tier limits

## Risk Mitigation (Serverless-Specific)

### Technical Risks
- **Lambda cold starts**: Use provisioned concurrency for critical functions
- **Free tier exhaustion**: CloudWatch alarms at 80% usage thresholds
- **Rate limiting by targets**: SQS with exponential backoff, respect robots.txt
- **DynamoDB throttling**: On-demand billing with burst capacity monitoring
- **Lambda timeout limits**: Split long operations into Step Functions workflows

### Cost Risks
- **Unexpected AWS charges**: Billing alarms, reserved concurrency limits
- **Claude API costs**: Prompt caching, usage monitoring, rate limiting
- **Data storage growth**: S3 lifecycle policies, DynamoDB TTL implementation
- **Free tier graduation**: Gradual scaling plan, cost optimization reviews

### Legal & Compliance Risks  
- **Web scraping legality**: Legal review, fair use practices, robots.txt compliance
- **Data privacy**: GDPR-compliant data deletion, anonymization processes
- **API terms violations**: Regular compliance audits of third-party services

### Business Risks
- **Serverless vendor lock-in**: Abstract AWS services behind interfaces
- **Competition**: Focus on unique intelligence insights, rapid iteration
- **Market fit**: MVP validation, user feedback integration

## Success Metrics (Serverless KPIs)

### Technical KPIs
- Data collection success rate (>95%)
- Lambda cold start impact (<5% of requests)
- Step Functions workflow completion rate (>98%)
- API response times (<300ms for cached, <2s for new requests)
- AWS Free Tier utilization (<80% to avoid overages)

### Cost KPIs
- Monthly AWS costs within budget ($60-350)
- Claude API cost per report (<$2)
- Cost per successful company research (<$5)
- Free tier graduation timeline (target: 6+ months)

### Business KPIs
- User research completion rate
- Report quality scores (user ratings)
- Time savings vs manual research
- Lead conversion improvement metrics
- User retention and platform adoption

## Immediate Next Steps (Week 1)

1. **AWS Account Setup & Billing Alerts**
   - Create AWS account with free tier
   - Set up billing alarms at 80% thresholds
   - Configure IAM users and roles

2. **Repository & IaC Setup**
   - Initialize Git repository
   - Choose AWS CDK vs Serverless Framework
   - Create basic project structure

3. **DynamoDB Schema Design**
   - Finalize single-table design
   - Plan GSI access patterns
   - Create initial data models

4. **API Design & Documentation**
   - Define REST API endpoints
   - Create OpenAPI specification
   - Plan authentication flows

5. **Frontend Architecture Decision**
   - React vs Vue.js selection
   - Static site generation approach
   - UI component library choice

---

*This revised plan optimizes for AWS Free Tier deployment with a serverless-first approach. The architecture minimizes costs while maximizing scalability and maintainability. Regular cost monitoring and optimization will be essential as the platform grows.*
