# Open-CCMD: Open Source CLAUDE.md Files

> **Making affordable AI models perform like Claude Code + Sonnet**

Open-CCMD is a collection of expert-level coding assistant guides (CLAUDE.md files) for different programming languages and frameworks. These guides help open-source AI models achieve better performance in code generation, debugging, and development tasks.

## 🎯 Problem Statement

Open-source AI models are affordable but often struggle with code quality and debugging efficiency. This project provides comprehensive, language-specific guides that help these models:

- ✅ Write better, more maintainable code
- ✅ Debug more efficiently with systematic approaches
- ✅ Follow best practices and patterns
- ✅ Understand codebases before making changes
- ✅ Test and verify changes properly

**Goal**: Make cheap open-source models perform on par with Claude Code + Sonnet for coding tasks.

## 📚 Available Guides

| Language/Framework | File | Status |
|-------------------|------|--------|
| C | [`languages/C_CLAUDE.md`](languages/C_CLAUDE.md) | ✅ Complete |
| C# | [`languages/CSHARP_CLAUDE.md`](languages/CSHARP_CLAUDE.md) | ✅ Complete |
| C++ | [`languages/CPP_CLAUDE.md`](languages/CPP_CLAUDE.md) | ✅ Complete |
| Go | [`languages/GO_CLAUDE.md`](languages/GO_CLAUDE.md) | ✅ Complete |
| Java | [`languages/JAVA_CLAUDE.md`](languages/JAVA_CLAUDE.md) | ✅ Complete |
| JavaScript | [`languages/JAVASCRIPT_CLAUDE.md`](languages/JAVASCRIPT_CLAUDE.md) | ✅ Complete |
| Python | [`languages/PYTHON_CLAUDE.md`](languages/PYTHON_CLAUDE.md) | ✅ Complete |
| Rust | [`languages/RUST_CLAUDE.md`](languages/RUST_CLAUDE.md) | ✅ Complete |
| TypeScript | [`languages/TYPESCRIPT_CLAUDE.md`](languages/TYPESCRIPT_CLAUDE.md) | ✅ Complete |

*More languages and frameworks coming soon! See [Contributing](#contributing) to add your own.*

## 🚀 Quick Start

### For Users

1. **Download the guide** for your language:
   ```bash
   # Clone the repository
   git clone https://github.com/yourusername/open-ccmd.git
   cd open-ccmd
   
   # Copy the guide to your project
   cp languages/PYTHON_CLAUDE.md /path/to/your/project/CLAUDE.md
   ```

2. **Use with your AI assistant**:
   - Add the guide to your project root as `CLAUDE.md`
   - Reference it in your AI assistant's system prompt or context
   - The guide will help the model follow best practices and systematic approaches

3. **Customize** (optional):
   - Add project-specific patterns or conventions
   - Include framework-specific guidelines
   - Add your team's coding standards

### For AI Assistants

When working with a project that includes a `CLAUDE.md` file:

1. **Read the guide first** - Understand the methodology and principles
2. **Follow Phase 0** - Always start with the 5-Question Framework
3. **Apply the workflows** - Use the systematic approaches for coding, debugging, and testing
4. **Verify everything** - Never assume code works; always test and verify

## 📖 What's in Each Guide?

Each language-specific guide includes:

- **🎯 Phase 0: Strategic Analysis** - The 5-Question Framework for understanding tasks
- **🔌 Interface-First Design** - Define contracts before implementation
- **🏗️ Bottom-Up Development** - Build and test incrementally
- **✋ Verification Protocol** - Never assume, always verify
- **📐 Syntax & Type Checking** - Catch errors early
- **🔍 Hypothesis-Driven Debugging** - Systematic problem-solving
- **🚨 Common Failure Patterns** - Learn from mistakes
- **📚 Language-Specific Patterns** - Best practices for the language

## 🏗️ Project Structure

```
open-ccmd/
├── languages/              # Language-specific guides
│   ├── C_CLAUDE.md
│   ├── CPP_CLAUDE.md
│   ├── CSHARP_CLAUDE.md
│   ├── GO_CLAUDE.md
│   ├── JAVA_CLAUDE.md
│   ├── JAVASCRIPT_CLAUDE.md
│   ├── PYTHON_CLAUDE.md
│   ├── RUST_CLAUDE.md
│   ├── TYPESCRIPT_CLAUDE.md
│   └── [More languages...]
├── CONTRIBUTING.md        # How to contribute
├── LICENSE                # MIT License
└── README.md              # This file
```

## 🤝 Contributing

We welcome contributions! Whether you want to:

- Add a new language or framework guide
- Improve existing guides
- Fix bugs or typos
- Add examples or use cases

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

### Quick Contribution Steps

1. Fork the repository
2. Create a new guide in `languages/` following the structure of existing guides
3. Test it with an AI assistant
4. Submit a pull request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Inspired by Claude Code's systematic approach to coding assistance
- Built for the open-source AI community
- Contributions from developers who want better AI coding tools

## 🔮 Roadmap

- [x] Add guides for TypeScript/JavaScript
- [x] Add guides for Go, Rust, C++
- [x] Add guide for C
- [x] Add guide for C#
- [ ] Add framework-specific guides (React, Django, Spring Boot, etc.)

## 💬 Discussion

Have questions, suggestions, or want to discuss improvements? 

- Open an issue on GitHub
- Start a discussion
- Contribute a guide!

## ⭐ Star History

If you find this project useful, please consider giving it a star! It helps others discover the project.

---

**Remember**: *"Simple, working code beats complex, perfect code."*

**Think strategically. Act systematically. Iterate until it works. Always clean up.**

