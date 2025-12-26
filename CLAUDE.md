# CLAUDE.md - AI Assistant Guide for aws_sample_codedeploy

This document provides comprehensive guidance for AI assistants working with this AWS CodeDeploy sample repository.

## Repository Overview

**Purpose**: Sample AWS CodeDeploy project demonstrating automated deployment of a simple web application
**Language**: Japanese (documentation and content)
**Platform**: AWS CodeDeploy on Linux
**Application Type**: Static HTML page served via Apache HTTP Server (httpd)

## Codebase Structure

```
aws_sample_codedeploy/
├── appspec.yml              # AWS CodeDeploy application specification
├── index.html               # Sample web application (deployment artifact)
├── scripts/                 # Deployment lifecycle hook scripts
│   ├── install_dependencies # Installs Apache (httpd)
│   ├── start_server        # Starts Apache service
│   └── stop_server         # Stops Apache service
├── LICENSE.txt             # License information
└── README.md               # Project README (Japanese)
```

## Key Files and Their Purposes

### appspec.yml
**Location**: `/appspec.yml`
**Purpose**: AWS CodeDeploy configuration file that defines deployment behavior

**Key sections**:
- `version`: AppSpec file format version (0.0)
- `os`: Target operating system (linux)
- `files`: Defines source files and their deployment destinations
  - Copies `index.html` to `/var/www/html/` (Apache's default document root)
- `hooks`: Lifecycle event hooks with execution order:
  1. **ApplicationStop**: `scripts/stop_server` - Stops httpd if running
  2. **BeforeInstall**: `scripts/install_dependencies` - Installs httpd
  3. **BeforeInstall**: `scripts/start_server` - Starts httpd

**Important**: All scripts run as `root` with a 300-second timeout.

### index.html
**Location**: `/index.html`
**Purpose**: Sample web page deployed to demonstrate successful deployment
**Content**: Japanese congratulations page with AWS CodeDeploy branding
**Deployment Target**: `/var/www/html/index.html`

### Deployment Scripts

#### scripts/install_dependencies
**Purpose**: Installs Apache HTTP Server
**Command**: `yum install -y httpd`
**Execution Phase**: BeforeInstall hook

#### scripts/start_server
**Purpose**: Starts Apache HTTP Server
**Command**: `service httpd start`
**Execution Phase**: BeforeInstall hook (runs after install_dependencies)

#### scripts/stop_server
**Purpose**: Gracefully stops Apache if it's running
**Logic**:
- Checks if httpd process exists using `pgrep httpd`
- Stops service if process is found
**Execution Phase**: ApplicationStop hook

**Note**: There's a bash syntax issue on line 2: `isExistApp = \`pgrep httpd\`` has spaces around `=` which will cause an error. Correct syntax should be `isExistApp=\`pgrep httpd\`` (no spaces).

## Development Workflows

### AWS CodeDeploy Deployment Lifecycle

When AWS CodeDeploy executes a deployment, the following sequence occurs:

1. **ApplicationStop** → `scripts/stop_server`
   - Stops existing httpd service if running

2. **DownloadBundle** (automatic)
   - CodeDeploy downloads deployment package

3. **BeforeInstall** → `scripts/install_dependencies`
   - Installs httpd package

4. **Install** (automatic)
   - Copies `index.html` to `/var/www/html/`

5. **BeforeInstall** → `scripts/start_server`
   - Starts httpd service

6. **ApplicationStart** (not used in this project)

7. **ValidateService** (not used in this project)

### Git Workflow

**Current Branch**: `claude/add-claude-documentation-HDGY2`
**Commit History**:
- fb17310: Merge pull request #2 from m19cmjigen/issue-1
- 0cf6d7c: テンプレートを変更しました #1
- 28f94af: initinal commit.
- 714fb35: Initial commit

**Branch Naming Convention**: Feature branches use `claude/` prefix

## Key Conventions for AI Assistants

### Language and Localization
- **Primary Language**: Japanese
- **Code Comments**: Minimal (scripts are self-documenting)
- **User-Facing Content**: Japanese (see index.html)
- When modifying user-facing content, maintain Japanese language unless explicitly requested otherwise

### Script Conventions
- **Shell**: Bash scripts with `#!/bin/bash` shebang
- **Line Endings**: Unix-style (LF)
- **Permissions**: Scripts should be executable
- **Error Handling**: Minimal; relies on AWS CodeDeploy's timeout mechanism
- **Package Manager**: Uses `yum` (Amazon Linux/RHEL-based)

### AWS CodeDeploy Best Practices for This Repository

1. **Script Idempotency**: Scripts should handle repeated executions gracefully
   - `install_dependencies`: `yum install -y` handles already-installed packages
   - `stop_server`: Checks if process exists before attempting stop

2. **Timeout Awareness**: All hooks have 300-second timeout
   - Keep operations within this limit
   - Long-running operations should provide feedback

3. **Error Propagation**: Non-zero exit codes cause deployment failure
   - Ensure scripts exit cleanly on success
   - Fix the bash syntax error in `stop_server`

### Code Quality Issues to Address

**Known Issue in scripts/stop_server**:
```bash
# INCORRECT (line 2):
isExistApp = `pgrep httpd`

# CORRECT:
isExistApp=`pgrep httpd`
```
Bash variables cannot have spaces around the `=` operator.

## Making Changes to This Repository

### Before Modifying Files

1. **Read First**: Always read files before modifying them
2. **Understand Context**: Review appspec.yml to understand deployment flow
3. **Test Locally**: If possible, test scripts in a Linux environment
4. **Preserve Language**: Maintain Japanese in user-facing content

### Common Modification Scenarios

#### Adding New Deployment Files
1. Add file to repository root or appropriate directory
2. Update `appspec.yml` files section:
```yaml
files:
  - source: /your-new-file
    destination: /target/path/
```

#### Adding New Lifecycle Hooks
1. Create script in `scripts/` directory
2. Make executable: `chmod +x scripts/your-script`
3. Update `appspec.yml` hooks section with appropriate event name
4. Available hooks: BeforeInstall, AfterInstall, ApplicationStart, ApplicationStop, ValidateService

#### Modifying Deployment Scripts
1. Read existing script first
2. Maintain bash shebang
3. Keep root execution context in mind
4. Test for idempotency
5. Ensure proper error handling (exit codes)

### Deployment Testing

This repository is designed for AWS CodeDeploy. To test:

1. **Prerequisites**:
   - AWS account with CodeDeploy configured
   - EC2 instance(s) with CodeDeploy agent installed
   - Appropriate IAM roles and permissions

2. **Deployment Package**:
   - Create zip/tar of repository contents
   - Upload to S3 or GitHub
   - Configure CodeDeploy application and deployment group

3. **Validation**:
   - After deployment, verify httpd is running: `service httpd status`
   - Check web page: `curl http://localhost/index.html`
   - Verify file ownership and permissions

## Architecture and Design Decisions

### Why Apache HTTP Server?
- Simple, widely-supported web server
- Available in default yum repositories
- Minimal configuration required for static content
- Standard choice for Amazon Linux deployments

### Why These Lifecycle Hooks?
- **ApplicationStop first**: Ensures clean shutdown of previous version
- **BeforeInstall for dependencies**: Installs server before file copy
- **BeforeInstall for start**: Starts service after files are in place
- **No AfterInstall/ValidateService**: Simple application doesn't require validation

### File Placement
- `/var/www/html/`: Apache's default DocumentRoot on RHEL-based systems
- Allows immediate serving without additional configuration

## Security Considerations

1. **Root Execution**: All scripts run as root
   - Necessary for package installation and service management
   - Be cautious with user input or external data
   - Validate all operations

2. **No Input Validation**: Scripts don't accept user input
   - Reduces attack surface
   - Keep scripts simple and focused

3. **Package Installation**: Uses yum from default repositories
   - Trusted source for packages
   - No version pinning (accepts latest from repo)

## Troubleshooting Guide

### Common Issues

**Deployment fails at ApplicationStop**:
- Check syntax in `stop_server` script (spaces around `=`)
- Verify bash is available at `/bin/bash`
- Check CodeDeploy agent logs: `/var/log/aws/codedeploy-agent/`

**httpd fails to install**:
- Verify yum repository configuration
- Check network connectivity from EC2 instance
- Ensure sufficient disk space

**Service fails to start**:
- Check for port 80 conflicts: `netstat -tlnp | grep :80`
- Review httpd error logs: `/var/log/httpd/error_log`
- Verify SELinux permissions if enabled

**Files not copied**:
- Verify source paths in appspec.yml are correct (relative to repo root)
- Check destination directory exists and is writable
- Review CodeDeploy deployment logs

## Repository Metadata

**Created**: Initial commit on 714fb35
**Last Updated**: Latest commit fb17310 (Merge PR #2)
**Primary Contributors**: m19cmjigen
**License**: See LICENSE.txt
**Repository Type**: Sample/Tutorial

## AI Assistant Recommendations

When working with this repository:

1. **Respect the simplicity**: This is a tutorial/sample project - don't over-engineer
2. **Maintain Japanese content**: Unless explicitly requested to change
3. **Fix known issues**: The bash syntax error in `stop_server` should be corrected
4. **Test assumptions**: When modifying scripts, consider Amazon Linux environment
5. **Document changes**: Use clear commit messages, preferably in Japanese to match existing style
6. **Preserve appspec.yml structure**: AWS CodeDeploy is strict about format
7. **Consider backwards compatibility**: Changes should work with existing AWS CodeDeploy setup

## Related Resources

- [AWS CodeDeploy Documentation](https://docs.aws.amazon.com/codedeploy/)
- [AppSpec File Reference](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html)
- [CodeDeploy Lifecycle Events](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html)

---

**Document Version**: 1.0
**Last Updated**: 2025-12-26
**Maintained By**: AI Assistant Analysis
