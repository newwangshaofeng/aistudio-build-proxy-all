# Docker Build Instructions

## Issues Fixed

1. **Syntax Error**: Removed incomplete `ENV` statements that caused "can't find = in /bin/sh" error
2. **GitHub API Rate Limit**: Added `GITHUB_TOKEN` support to avoid 403 rate limit errors during `camoufox fetch`

## Building the Docker Image

### Option 1: With GitHub Token (Recommended)

To avoid GitHub API rate limits, set your GitHub token as an environment variable:

**Windows PowerShell:**
```powershell
$env:GITHUB_TOKEN="your_github_token_here"
docker-compose build
```

**Linux/Mac:**
```bash
export GITHUB_TOKEN="your_github_token_here"
docker-compose build
```

### Option 2: Without GitHub Token

If you don't have a GitHub token or want to try without it:

```powershell
docker-compose build
```

Note: This may fail with a 403 rate limit error if GitHub's API limit is exceeded.

## Getting a GitHub Token

1. Go to https://github.com/settings/tokens
2. Click "Generate new token" → "Generate new token (classic)"
3. Give it a name (e.g., "Docker Build")
4. Select scopes: No special scopes needed for public repo access
5. Click "Generate token"
6. Copy the token and use it as shown above

## Running the Container

```powershell
docker-compose up -d
```

## Troubleshooting

If the build still fails with rate limit errors:
1. Wait a few minutes and try again
2. Use a GitHub token (see above)
3. The Dockerfile has a retry mechanism (5 attempts with increasing delays)
