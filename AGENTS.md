# AGENTS.md

## Cursor Cloud specific instructions

This repository (`aws-admin-cursor`) is an AWS administration workspace. The AWS account (813297319090) has three existing Lambda functions (`set_thermo`, `set_lock_codes`, `listing_assist`) running Python 3.11.

### Installed tools

| Tool | Version | Notes |
|------|---------|-------|
| AWS CLI v2 | 2.34.x | Credentials via env vars (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`) |
| SAM CLI | 1.161.x | Installed via pip to `~/.local/bin` |
| AWS CDK | 2.x | Installed to `~/.npm-global/bin` |
| Node.js | 22.x | via nvm |
| TypeScript | 6.x | Global install |
| esbuild | 0.28.x | Global install |
| Python | 3.12 | System Python |
| cfn-lint | 1.51.x | CloudFormation linter |
| Docker | 28.5.x | fuse-overlayfs storage driver (nested container env) |
| jq | 1.7 | JSON processor |

### PATH setup

Both `~/.local/bin` (pip installs) and `~/.npm-global/bin` (npm global installs) must be on PATH. The update script handles this. If running interactively:

```
export PATH="$HOME/.npm-global/bin:$HOME/.local/bin:$PATH"
```

### Docker caveats

- Docker runs inside a nested container (Firecracker VM). The host cgroup is in **threaded mode**, so `sam local invoke` / `sam local start-api` fail with a cgroup error when trying to start Lambda emulation containers.
- **Workaround**: Invoke Lambda handlers directly with Node.js/Python, or invoke remotely with `aws lambda invoke`.
- Docker itself works for building images; containers that don't need domain cgroup controllers also work (e.g. `docker run --privileged --cgroupns=host alpine echo ok`).
- Docker daemon must be started manually: `sudo dockerd &>/tmp/dockerd.log &` then `sudo chmod 666 /var/run/docker.sock`.

### nvm warning

The npm global prefix (`~/.npm-global`) triggers a benign nvm warning. It does not affect functionality. Suppress with `nvm use --delete-prefix v22.22.2 --silent` if needed.

### Common commands

```bash
# List Lambda functions
aws lambda list-functions --output table

# Invoke a Lambda
aws lambda invoke --function-name <name> --payload '{}' --cli-binary-format raw-in-base64-out /tmp/response.json

# Validate SAM template
sam validate

# Build SAM project
sam build

# Lint CloudFormation
cfn-lint template.yaml

# CDK synth
cdk synth

# Check AWS identity
aws sts get-caller-identity
```
