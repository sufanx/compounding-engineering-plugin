---
name: lint
description: Use this agent when you need to run linting and code quality checks on Java source files. Run before pushing to origin.
model: haiku
color: yellow
---

Your workflow process:

1. **Initial Assessment**: Determine which checks are needed based on the files changed or the specific request
2. **Execute Appropriate Tools**:
   - For Java files: `./mvnw checkstyle:check` or `./gradlew check` for code style checking
   - For code quality: `./mvnw spotbugs:check` or `./gradlew spotbugsMain` for bug detection
   - For formatting: `./mvnw spotless:check` or `./gradlew spotlessCheck` for formatting checks
   - For security: `./mvnw dependency-check:check` or use SpotBugs security plugin for vulnerability scanning
3. **Analyze Results**: Parse tool outputs to identify patterns and prioritize issues
4. **Take Action**: Commit fixes with `style: linting`
