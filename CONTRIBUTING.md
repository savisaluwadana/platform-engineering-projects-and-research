# Contributing to Platform Engineering Projects and Research

Thank you for your interest in contributing! This repository is built by the community, for the community. Your contributions help aspiring platform engineers around the world.

## 🌟 How You Can Contribute

### 1. Add Project Ideas
Have a great project idea that helped you learn? Share it!
- Add it to the appropriate module's README
- Include clear objectives and learning outcomes
- Specify difficulty level (Beginner/Intermediate/Advanced)

### 2. Share Project Implementations
Built one of the projects? Share your implementation!
- Create a new folder under the module (e.g., `01-linux-fundamentals/projects/your-project-name`)
- Include complete source code
- Add thorough documentation
- Provide setup instructions

### 3. Improve Documentation
Help make the content clearer and more comprehensive:
- Fix typos and grammar
- Add explanations for complex topics
- Create diagrams and visuals
- Translate content to other languages

### 4. Add Resources
Found helpful resources? Share them!
- Add links to tutorials, articles, videos
- Include book recommendations
- Share certification study guides
- List useful tools and utilities

### 5. Share Learning Experiences
Document your journey:
- Write about challenges you faced
- Share solutions to common problems
- Create step-by-step guides
- Add troubleshooting tips

## 📋 Contribution Guidelines

### Project Submissions

When submitting a project implementation:

```
module-name/
└── projects/
    └── your-project-name/
        ├── README.md          # Project overview and setup
        ├── src/               # Source code
        ├── tests/             # Test files
        ├── docs/              # Additional documentation
        ├── examples/          # Usage examples
        └── assets/            # Images, diagrams, etc.
```

**README.md should include**:
- Project description and objectives
- Prerequisites
- Setup and installation instructions
- Usage examples
- Architecture/design decisions
- Technologies used
- Challenges and learnings
- Screenshots/demos (if applicable)
- License information

### Code Quality Standards

- **Clean Code**: Follow language-specific best practices
- **Documentation**: Comment complex logic, include docstrings
- **Testing**: Include unit tests where applicable
- **Security**: No hardcoded credentials or sensitive data
- **Linting**: Run linters before submission
- **Formatting**: Use consistent code formatting

### Documentation Standards

- **Clarity**: Write clear, concise explanations
- **Examples**: Include practical examples
- **Formatting**: Use proper Markdown formatting
- **Links**: Ensure all links are valid
- **Images**: Optimize images for web (<500KB)

## 🚀 Getting Started

### Setting Up Your Development Environment

1. **Fork the Repository**
   ```bash
   # Click 'Fork' on GitHub, then clone your fork
   git clone https://github.com/YOUR_USERNAME/platform-engineering-projects-and-research.git
   cd platform-engineering-projects-and-research
   ```

2. **Create a Branch**
   ```bash
   git checkout -b feature/your-contribution-name
   ```

3. **Make Your Changes**
   - Add your content
   - Test thoroughly
   - Update documentation

4. **Commit Your Changes**
   ```bash
   git add .
   git commit -m "Add: brief description of your contribution"
   ```

5. **Push to Your Fork**
   ```bash
   git push origin feature/your-contribution-name
   ```

6. **Create a Pull Request**
   - Go to your fork on GitHub
   - Click 'New Pull Request'
   - Provide a clear description of your changes

## 💡 Contribution Ideas

### For Beginners
- Fix typos and formatting issues
- Add clarifying examples
- Test and document project setups
- Add screenshots to existing projects

### For Intermediate Contributors
- Implement project ideas
- Add missing theory sections
- Create tutorial videos
- Write blog posts about projects

### For Advanced Contributors
- Design new project ideas
- Review and improve existing projects
- Create comprehensive guides
- Mentor other contributors

## 📝 Pull Request Process

1. **Before Submitting**
   - Test your changes thoroughly
   - Update relevant documentation
   - Check for typos and formatting
   - Ensure no sensitive data is included

2. **PR Description Should Include**
   - What changes were made
   - Why these changes are needed
   - Any breaking changes
   - Screenshots (if UI changes)
   - Testing performed

3. **Review Process**
   - Maintainers will review your PR
   - Address any requested changes
   - Once approved, your PR will be merged
   - You'll be added to contributors list!

## 🏆 Recognition

Contributors are recognized in several ways:
- Listed in the README.md contributors section
- Mentioned in release notes
- Featured in community highlights
- Potential collaboration opportunities

## ❓ Questions?

- **General Questions**: Open a GitHub Discussion
- **Bug Reports**: Open a GitHub Issue
- **Feature Requests**: Open a GitHub Issue
- **Security Issues**: Email directly (see SECURITY.md)

## 📄 Code of Conduct

### Our Standards

- **Be Respectful**: Treat everyone with respect
- **Be Collaborative**: Work together constructively
- **Be Inclusive**: Welcome diverse perspectives
- **Be Patient**: Help others learn
- **Be Professional**: Keep discussions focused and productive

### Unacceptable Behavior

- Harassment or discrimination
- Trolling or insulting comments
- Personal or political attacks
- Publishing others' private information
- Any conduct inappropriate in a professional setting

### Enforcement

Violations may result in:
- Warning from maintainers
- Temporary ban from contributions
- Permanent ban from the project

## 📊 Contribution Types

### 🐛 Bug Fixes
- Documentation errors
- Broken links
- Code issues in examples
- Setup/installation problems

### ✨ Enhancements
- New project ideas
- Additional resources
- Improved explanations
- Better examples

### 📚 Documentation
- New guides and tutorials
- Architecture diagrams
- Video tutorials
- Translations

### 🎨 Design
- Improve README layouts
- Create visual assets
- Design diagrams
- Improve user experience

## 🎯 Current Priorities

Check the [GitHub Issues](https://github.com/savisaluwadana/platform-engineering-projects-and-research/issues) for current priorities. Look for:
- Issues labeled `good first issue`
- Issues labeled `help wanted`
- Issues labeled `documentation`

## 📖 Style Guide

### Markdown
```markdown
# H1 for main titles
## H2 for sections
### H3 for subsections

- Use bullets for lists
- Use **bold** for emphasis
- Use `code` for commands and code snippets
- Use ```language for code blocks
```

### Commit Messages
```
Type: Brief description (50 chars max)

Detailed explanation if needed (wrap at 72 chars)

- Bullet points for multiple changes
- Reference issues: Fixes #123

Types: Add, Update, Fix, Remove, Refactor, Docs
```

### Code Comments
```python
# Good: Explains WHY
# Using exponential backoff to handle rate limiting

# Bad: Explains WHAT (code already shows this)
# Set retry to 3
```

## 🔄 Keeping Your Fork Updated

```bash
# Add upstream remote
git remote add upstream https://github.com/savisaluwadana/platform-engineering-projects-and-research.git

# Fetch upstream changes
git fetch upstream

# Merge upstream changes
git checkout main
git merge upstream/main

# Push to your fork
git push origin main
```

## 📧 Contact

- **GitHub Issues**: For bugs and features
- **GitHub Discussions**: For questions and ideas
- **Pull Requests**: For contributions

## 🙏 Thank You!

Every contribution, no matter how small, makes a difference. Thank you for helping make platform engineering education accessible to everyone!

---

**Happy Contributing! 🚀**
