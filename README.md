# Production Repository (prod-repo)

## Overview
Production repository containing stable, QA-approved code deployed to live environments. This is the final stage of the CI/CD pipeline.

## Purpose
- Production-ready code storage
- Live environment deployment
- Production Docker image builds
- Server deployment via SSH
- Production monitoring and maintenance

## Workflow

### Manual Deployment from Test
Code arrives here only after QA approval:
1. QA team validates code in `test-repo`
2. Manual workflow is triggered in `test-repo`
3. Code is synced to this repository (excluding `.github/workflows/` and `README.md`)
4. Production deployment workflow triggers automatically
5. Docker image is built and pushed to Docker Hub
6. Application is deployed to production server via SSH

## Repository Structure
```
prod-repo/
├── .github/
│   └── workflows/
│       └── deploy-production.yml  # Builds & deploys to production
├── app.js                          # Synced from test-repo
├── public/                         # Synced from test-repo
├── tests/                          # Synced from test-repo
├── Dockerfile                      # Synced from test-repo
├── package.json                    # Synced from test-repo
└── README.md                       # This file (not synced)
```

## Production Deployment

### Automatic Triggers
Deployment to production servers happens automatically when:
- Code is pushed to `main` branch (via manual deployment from test-repo)
- All tests pass in Docker build
- Docker image successfully pushed to Docker Hub

### Deployment Steps
1. **Build Phase**
   - Multi-stage Docker build
   - Run test suite
   - Create production image
   - Tag with commit SHA and latest

2. **Push Phase**
   - Authenticate with Docker Hub
   - Push image to registry
   - Update latest tag

3. **Deploy Phase**
   - SSH into production server
   - Pull latest Docker image
   - Stop existing container
   - Start new container
   - Verify health check

## CI/CD Pipeline
**Current Position**: END of pipeline (PRODUCTION)

```
[dev-repo] --automatic--> [test-repo] --manual--> [prod-repo]
                                                   YOU ARE HERE
```

### Previous Stage
- Repository: `test-repo`
- Deployment: Manual after QA approval

### This is Production
- All code here has been tested and approved
- Automatic deployment to live servers
- Zero-downtime deployment strategy

## Production Server

### Access
Production server access is restricted to:
- DevOps team
- System administrators
- On-call engineers (emergency only)

### SSH Deployment
The deployment workflow uses SSH to:
```bash
# Pull latest image
docker pull username/app-name:latest

# Stop old container
docker stop app-name || true
docker rm app-name || true

# Start new container
docker run -d \
  --name app-name \
  -p 3000:3000 \
  --restart unless-stopped \
  username/app-name:latest
```

## Docker Hub

### Image Repository
- Registry: Docker Hub
- Repository: `username/app-name`
- Tags:
  - `latest` - Most recent production build
  - `<commit-sha>` - Specific version tags

### Pulling Production Image
```bash
docker pull username/app-name:latest
```

## Monitoring

### Health Checks
- Endpoint: `GET /health`
- Expected Response: `{ "status": "healthy" }`
- Check Interval: 30 seconds

### Key Metrics to Monitor
- Application uptime
- Response times
- Error rates
- Memory usage
- CPU usage
- Request volume

### Logging
- Application logs available via Docker
- View logs: `docker logs app-name`
- Follow logs: `docker logs -f app-name`

## Environment-Specific Files
The following files are **NOT** synced from upstream:
- `.github/workflows/*` - Production-specific workflow
- `README.md` - This production documentation

## Production Best Practices

### ✅ DO
- Monitor application health continuously
- Review deployment logs after each release
- Keep Docker images updated
- Maintain backup and rollback procedures
- Document all production changes
- Test rollback procedures regularly

### ❌ DON'T
- Never commit directly to this repository
- Never skip the test-repo stage
- Never deploy during peak hours without approval
- Never expose production credentials
- Never disable health checks

## Secrets Required
- `DOCKER_HUB_USERNAME` - Docker Hub username
- `DOCKER_HUB_PASSWORD` - Docker Hub password/token
- `SERVER_SSH_KEY` - SSH private key for production server
- `SERVER_HOST` - Production server hostname/IP
- `SERVER_USER` - SSH user for deployment

## Rollback Procedure

### Quick Rollback
If issues are detected in production:

1. **Find Previous Working Version**
   ```bash
   docker images username/app-name
   ```

2. **Stop Current Container**
   ```bash
   docker stop app-name
   docker rm app-name
   ```

3. **Start Previous Version**
   ```bash
   docker run -d \
     --name app-name \
     -p 3000:3000 \
     --restart unless-stopped \
     username/app-name:<previous-commit-sha>
   ```

4. **Verify Application**
   ```bash
   curl http://localhost:3000/health
   ```

### Rollback via Git
If a complete rollback is needed:
1. Identify the last working commit
2. Create a revert commit
3. Redeploy from test-repo

## Incident Response

### P1 - Critical (Production Down)
1. Immediately rollback to last known good version
2. Notify on-call team
3. Investigate root cause
4. Document incident

### P2 - Major (Degraded Performance)
1. Monitor metrics closely
2. Assess impact
3. Decide: rollback or continue monitoring
4. Plan fix deployment

### P3 - Minor (Non-Critical Issue)
1. Document the issue
2. Schedule fix in next deployment cycle
3. Continue monitoring

## Deployment History
Track all production deployments with:
- Date and time
- Deployed by (GitHub actor)
- Commit SHA
- Deployment message
- Rollback performed (if any)

## Support

### Emergency Contact
- On-Call Engineer: [Contact info]
- DevOps Team: [Contact info]
- System Administrator: [Contact info]

### Documentation
- Runbooks: [Link to runbooks]
- Architecture Docs: [Link to docs]
- API Documentation: [Link to API docs]

## Maintenance Windows
- Preferred: Off-peak hours (2 AM - 5 AM UTC)
- Notification: 24 hours advance notice
- Duration: Target < 30 minutes downtime

---

**⚠️ PRODUCTION ENVIRONMENT - HANDLE WITH CARE**
