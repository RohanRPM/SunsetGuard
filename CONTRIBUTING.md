# Contributing to SunsetGuard

🚧 **Note**: This project is currently in active development (Phase 0-8). Contributions will be fully open after v0.1.0 release (January 2027).

## Current Status

This is a learning project being built by [Rohan Meshram](https://github.com/rohanmeshram) to demonstrate backend system design skills. 

**During development (Phases 0-8):**
- The project is public to share progress
- Issues and suggestions are welcome
- Pull requests will be reviewed but may be delayed
- Focus is on completing the roadmap first

**After v0.1.0 (Phase 8 complete):**
- Full contribution guidelines will be added
- Good first issues will be labeled
- PR review process will be formalized

## How You Can Help Now

### 1. Star the Repository ⭐
Show support and help others discover the project.

### 2. Open Issues 🐛
Found a bug or have a feature idea? Open an issue!

**Bug reports should include:**
- What you expected to happen
- What actually happened
- Steps to reproduce
- Your environment (OS, Java version, Docker version)

**Feature requests should include:**
- The problem you're trying to solve
- Why existing features don't cover it
- Your proposed solution (optional)

### 3. Improve Documentation 📖
Spot a typo or confusing explanation? Documentation PRs are always welcome!

### 4. Share Feedback
Tried the project? Let me know:
- What worked well
- What was confusing
- What's missing

## Development Setup

See [docs/SETUP.md](docs/SETUP.md) for complete environment setup instructions.

## Code Style

This project follows standard Java conventions:
- Use **4 spaces** for indentation (not tabs)
- **120 characters** max line length
- **Lombok** for boilerplate reduction
- **Clear naming** over clever code
- **Comments** for why, not what

## Commit Message Format

```
<type>: <short summary>

<optional detailed description>

<optional footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring (no behavior change)
- `test`: Adding or updating tests
- `chore`: Build, dependencies, tooling

**Examples:**
```
feat: add verdict engine with confidence levels

Implements the safe-to-remove verdict logic that analyzes
usage patterns and data coverage.

Closes #42
```

```
fix: prevent cardinality explosion in endpoints table

Added cap of 1000 new endpoints per service per day.
Overflow routes are bucketed into '_other'.
```

## Testing

All new code should include tests:
- **Unit tests** for pure logic (verdict engine, aggregator)
- **Integration tests** for API endpoints (with Testcontainers)
- **Slice tests** for repository and controller layers

```bash
# Run all tests
mvn test

# Run specific test
mvn test -Dtest=VerdictEngineTest

# Run with coverage
mvn clean test jacoco:report
# Report at: target/site/jacoco/index.html
```

## Pull Request Process

**After v0.1.0:**

1. Fork the repository
2. Create a feature branch (`git checkout -b feat/amazing-feature`)
3. Make your changes
4. Add tests
5. Ensure tests pass (`mvn clean verify`)
6. Commit with clear messages
7. Push to your fork
8. Open a Pull Request

**PR checklist:**
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No breaking changes (or clearly marked)
- [ ] Follows code style
- [ ] Commit messages are clear

## Code of Conduct

Be respectful, inclusive, and constructive. This is a learning project and a welcoming space for developers of all experience levels.

**Expected behavior:**
- ✅ Helpful, constructive feedback
- ✅ Respectful disagreement
- ✅ Patience with questions
- ✅ Celebrating successes

**Not tolerated:**
- ❌ Harassment or discrimination
- ❌ Personal attacks
- ❌ Trolling or inflammatory comments
- ❌ Spam

Violations will result in being blocked from the project.

## Questions?

- **Documentation**: Check [README.md](README.md) and [docs/](docs/)
- **Setup issues**: See [docs/SETUP.md](docs/SETUP.md)
- **Feature questions**: Open a GitHub issue
- **General chat**: (Discord/Slack link will be added after v0.1.0)

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0. See [LICENSE](LICENSE).

---

**Thank you for your interest in SunsetGuard!** 🌅

This project is built with ❤️ and coffee ☕ to help teams safely deprecate APIs.
