# Contributing to Bluefin ISO

Thank you for your interest in contributing to Bluefin ISO! This document provides guidelines for contributing to this repository.

## Code of Conduct

Please be respectful and considerate in all interactions.

## Conventional Commits

This project uses [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/#specification) for all commit messages and pull request titles.

Format: `<type>(<scope>): <description>`

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `chore`: Maintenance tasks
- `ci`: CI/CD changes
- `refactor`: Code refactoring
- `test`: Test changes

Examples:
```
feat(iso): add support for ARM64 builds
fix(anaconda): correct flatpak installation script
docs(readme): update build instructions
ci(workflow): optimize ISO build matrix
```

## Development Workflow

1. **Fork the repository** and create a new branch
2. **Make your changes** following the guidelines below
3. **Test your changes** locally if possible
4. **Run validation** before committing:
   ```bash
   pre-commit run --all-files
   just check
   ```
5. **Commit your changes** using conventional commits
6. **Push to your fork** and create a pull request

## Making Changes

### ISO Configuration Changes
- Modify scripts in `iso_files/` carefully
- Test changes thoroughly as they affect the installation experience
- Document any new configuration options

### Flatpak List Changes
- Add one flatpak per line
- Include comments for non-obvious additions
- Ensure flatpaks exist on Flathub
- Run validation: The workflow will check flatpak validity

### Workflow Changes
- Test workflow changes in your fork first
- Use workflow dispatch for manual testing
- Consider resource usage (runner time, storage)
- Update documentation if adding new features

### Just Recipe Changes
- Follow existing recipe patterns
- Use `just check` to validate syntax
- Use `just fix` to auto-format
- Document parameters and usage

## Validation Requirements

All contributions must pass:

1. **Pre-commit hooks**
   ```bash
   pre-commit run --all-files
   ```
   Note: `.devcontainer.json` JSON check failure is expected

2. **Just syntax check** (if Just is available)
   ```bash
   just check
   ```

3. **Workflow validation**
   - GitHub Actions must pass
   - No breaking changes to existing workflows

## Best Practices

### Be Surgical
- Make minimal changes to achieve your goal
- Don't refactor unrelated code
- Keep commits focused and atomic

### Documentation
- Update README.md if adding features
- Add comments for complex configurations
- Update AGENTS.md if changing build patterns

### Testing
- Test ISO builds locally when possible
- Use workflow dispatch for CI testing
- Verify flatpak installations work correctly

## AI Attribution

If using AI tools for assistance, include attribution in your commit:

```
Assisted-by: [Model Name] via [Tool Name]
```

Example:
```
feat(iso): add new configuration option

Add support for custom kernel arguments

Assisted-by: Claude 3.5 Sonnet via GitHub Copilot
```

## Getting Help

- Review [AGENTS.md](AGENTS.md) for technical details
- Check existing issues and pull requests
- Ask questions in pull request discussions

## Pull Request Process

1. **Ensure validation passes**
2. **Update documentation** as needed
3. **Use conventional commit** format for PR title
4. **Describe your changes** clearly in PR description
5. **Link related issues** if applicable
6. **Wait for review** - maintainers will review and provide feedback

## Review Guidelines

Reviewers will check for:
- Conventional commit format
- Pre-commit validation passing
- Surgical, focused changes
- Proper documentation
- No breaking changes (unless intentional)

## Questions?

If you have questions about contributing, feel free to:
- Open an issue for discussion
- Ask in pull request comments
- Review the main Bluefin repository for additional context

Thank you for contributing to Bluefin ISO! 🎉
