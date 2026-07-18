# Contributing to Kali Linux Tools Documentation

Thanks for your interest in contributing! This project is actively developing and we welcome help.

## How to Contribute

### 1. Pick a Tool
Check the [README](README.md) for which phases are still developing. Pick any tool that needs documentation.

### 2. Follow the Template
Every tool must follow the 8-chapter structure:

```markdown
## Chapter 1: Introduction & Overview
- What is it?
- When to use / when NOT to use
- Key concepts

## Chapter 2: Installation & Setup
- System requirements
- Installation methods (APT, source, Docker)
- Verification

## Chapter 3: Basic Usage
- First run with sample output
- Essential commands with explanations
- Complete flag reference table

## Chapter 4: Intermediate Usage
- Scripting and automation (bash + python)
- Tool chaining

## Chapter 5: Advanced Usage
- Custom techniques
- Integration in pentest workflow

## Chapter 6: Real-World Scenarios
- Security audit walkthrough
- CTF challenge
- Bug bounty

## Chapter 7: Detection & Defense
- How defenders detect it
- Signatures and mitigation

## Chapter 8: Troubleshooting
- Common errors
- FAQ
```

### 3. Quality Standards
- Minimum 500 lines per tool
- Every command must have an `**Explanation:**` line
- Use real examples, not placeholders
- Include YAML frontmatter with all fields
- Create a CHEATSHEET.md alongside the main doc

### 4. Submit a PR
1. Fork the repo
2. Create a branch: `git checkout -b tool/tool-name`
3. Add your documentation
4. Submit a pull request with a clear description

## Questions?

Open an issue if you have questions or want to discuss what to work on next.
