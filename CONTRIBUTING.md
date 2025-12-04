# Contributing to Open-CCMD

Thank you for your interest in contributing to Open-CCMD! This guide will help you create high-quality language and framework guides.

## 🎯 What We're Looking For

We welcome contributions for:

- **New language guides** (Swift, Kotlin, PHP, Ruby, etc.)
- **Framework-specific guides** (React, Django, Spring Boot, etc.)
- **Improvements to existing guides** (better examples, clearer explanations)
- **Bug fixes and typos**
- **Documentation improvements**

## 📋 Creating a New Guide

### Step 1: Check if it Already Exists

Check the `languages/` directory to see if a guide for your language/framework already exists or is in progress.

### Step 2: Create Your Guide

1. Create a new file `languages/[LANGUAGE]_CLAUDE.md`
2. Follow the structure of existing guides (C, C#, C++, Go, Java, JavaScript, Python, Rust, TypeScript) as reference
3. Fill in language-specific content for each section

### Step 3: Language-Specific Sections

Each guide should include:

#### Required Sections:
- ✅ **Quick Start** - Overview of the methodology
- ✅ **Phase 0: Strategic Analysis** - 5-Question Framework
- ✅ **Interface-First Design** - Language-specific interface patterns
- ✅ **Bottom-Up Development** - Incremental building approach
- ✅ **Verification Protocol** - Testing and validation
- ✅ **Syntax & Type Checking** - Language-specific syntax rules
- ✅ **Error Prevention** - Common mistakes and how to avoid them
- ✅ **Decision Framework** - When to use which tools/approaches
- ✅ **Iterative Development** - Core development loop
- ✅ **Debugging** - Hypothesis-driven debugging
- ✅ **Tool Usage** - Language-specific tools and commands
- ✅ **Common Failure Patterns** - Real examples of failures and fixes
- ✅ **Project Specifics** - Language conventions, project structure, code style

#### Language-Specific Adaptations:

**Build System:**
- C: `gcc`, `clang`, `make`, `cmake`
- C#: `dotnet build`, `dotnet test`, `dotnet format`
- C++: `g++`, `clang++`, `make`, `cmake`
- Go: `go test`, `go build`, `gofmt`
- Java: `./mvnw`, `./gradlew`, `checkstyle`
- JavaScript: `npm`, `node`, `eslint`
- Python: `pytest`, `ruff`, `mypy`, `.venv`
- Rust: `cargo test`, `cargo build`, `rustfmt`
- TypeScript: `npm`, `tsc`, `eslint`

**Testing:**
- C: `make test` or custom test runner
- C#: `dotnet test --filter "FullyQualifiedName~TestClass.TestMethod"`
- C++: `make test` or `ctest` (CMake)
- Go: `go test ./...`
- Java: `./mvnw test -Dtest=TestClass#testMethod`
- JavaScript: `npm test` or `jest`
- Python: `pytest tests/test_file.py::test_function`
- Rust: `cargo test`
- TypeScript: `npm test` or `jest`

**Linting/Formatting:**
- C: `clang-format`, `cppcheck`
- C#: `dotnet format --verify`
- C++: `clang-format`, `cppcheck`
- Go: `gofmt -w .`
- Java: `./mvnw checkstyle:check`
- JavaScript: `npm run lint` or `eslint .`
- Python: `ruff check .`
- Rust: `cargo fmt`
- TypeScript: `npm run lint` or `eslint .`

### Step 4: Test Your Guide

Before submitting:

1. **Test with an AI assistant** - Use the guide with a model (like Ollama, LM Studio, etc.)
2. **Verify examples work** - All code examples should be syntactically correct
3. **Check consistency** - Follow the same structure and style as existing guides
4. **Proofread** - Check for typos, grammar, and clarity

### Step 5: Submit Your Contribution

1. Fork the repository
2. Create a branch: `git checkout -b add-[language]-guide`
3. Commit your changes: `git commit -m "Add [Language] guide"`
4. Push to your fork: `git push origin add-[language]-guide`
5. Open a Pull Request

## ✍️ Writing Guidelines

### Style and Tone

- **Clear and direct** - Write for AI assistants, be explicit
- **Action-oriented** - Use imperative mood ("Do this", "Check that")
- **Example-driven** - Show, don't just tell
- **Consistent formatting** - Use the same markdown structure as existing guides

### Code Examples

- ✅ Use real, working code examples
- ✅ Include both ❌ wrong and ✅ right examples
- ✅ Show complete, runnable examples when possible
- ✅ Use language-specific idioms and best practices
- ✅ Include error messages and how to fix them

### Structure Consistency

Follow this structure (same as all existing guides):

1. Title with language name
2. Core Philosophy
3. Quick Start
4. Phase 0: Strategic Analysis
5. Interface-First Design
6. Bottom-Up Development
7. Verification Protocol
8. Syntax & Type Checking
9. Error Prevention
10. Decision Framework
11. Iterative Development
12. Debugging
13. Tool Usage & Communication
14. Cognitive Strategies
15. File Management
16. Development Workflow
17. Quality & Definition of Done
18. Quick Reference Card
19. Common Failure Patterns
20. Success Criteria
21. Top Rules
22. Project Specifics

## 🔍 Review Process

When you submit a PR:

1. **Automated checks** - We'll verify the file structure
2. **Community review** - Other contributors will review
3. **Testing** - We'll test the guide with AI models
4. **Feedback** - We'll provide suggestions for improvement

## 📝 Checklist for New Guides

Before submitting, ensure:

- [ ] Guide follows the structure of existing guides
- [ ] All sections are filled with language-specific content
- [ ] Code examples are syntactically correct
- [ ] Build/test commands are accurate for the language
- [ ] Common patterns and idioms are included
- [ ] Error handling is language-appropriate
- [ ] File structure matches language conventions
- [ ] Guide has been tested with an AI assistant
- [ ] No typos or grammar errors
- [ ] Consistent formatting with other guides

## 🐛 Reporting Issues

Found a bug or have a suggestion? Open an issue with:

- **Type**: Bug, Enhancement, Documentation
- **Language**: Which guide is affected
- **Description**: Clear explanation of the issue
- **Example**: If applicable, include examples

## 💡 Ideas for Contributions

- Add more examples to existing guides
- Create framework-specific variants (e.g., `PYTHON_DJANGO_CLAUDE.md`)
- Add troubleshooting sections
- Create quick reference cards
- Add benchmarks or performance tips
- Translate guides to other languages (human languages)

## ❓ Questions?

- Open a discussion on GitHub
- Check existing issues and PRs
- Review existing guides for examples

Thank you for contributing to Open-CCMD! 🎉

