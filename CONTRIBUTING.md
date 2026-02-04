# Contributing to AIKIT Skill

## Repository Structure

```
goaikit/skill/
├── skill-project.toml       # Project-level configuration
└── aikit/                    # AIKIT skill
    ├── SKILL.md              # Skill documentation
    └── skill-project.toml    # Skill metadata
```

## Development

1. Edit files in `aikit/` subdirectory
2. Commit and push to `main` branch
3. The CI/CD workflow will automatically package and publish a new release when `aikit/` or the workflow file changes

## CI/CD

The GitHub Actions workflow in `.github/workflows/publish-skill.yml`:

- Triggers on push to `main` when paths `aikit/**` or `.github/workflows/publish-skill.yml` change, or on workflow_dispatch
- Packages the skill using FastSkill CLI
- Creates a GitHub release with the packaged skill artifact
- Uses a GitHub App token for release operations (delete old releases)

## Required Secrets

Configure these repository secrets for the publish workflow:

- `GH_APP_ID`: GitHub App ID for token generation
- `GH_APP_PRIVATE_KEY`: GitHub App private key for authentication
- `GITHUB_TOKEN`: Provided automatically by GitHub Actions

Ensure the GitHub App has **Repository → Contents: Read and write** on this repository.
