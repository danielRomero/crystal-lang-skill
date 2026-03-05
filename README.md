# Crystal Lang Skill

A comprehensive skill providing Crystal programming language guidance for AI agents. This skill helps AI agents generate idiomatic, type-safe Crystal code by providing best practices, patterns, and reference documentation.

## What is This?

Crystal Lang Skill is an opencode skill that equips AI agents with comprehensive knowledge about the Crystal programming language. It covers:

- Type system (inference, union types, generics)
- Performance patterns (structs vs classes, records)
- Object-oriented features (visibility, abstract classes, enums)
- Exception handling
- Concurrency (fibers, channels)
- Metaprogramming (macros)
- Code generation best practices

## Installation

### From GitHub

```bash
npx skills add danielRomero/crystal-lang-skill
```

### From URL

If hosting the skill yourself:

```bash
npx skills add https://your-domain.com/path/to/crystal-lang-skill/SKILL.md
```

### Local Development

To test the skill locally:

```bash
# Clone or copy this folder to your skills directory
# Then use the local path
npx skills add ./crystal-lang-skill
```

## Usage

Once installed, the skill activates automatically when you:

- Ask for help with Crystal code
- Need to generate Crystal code
- Work on Crystal projects (shards, apps, libraries)
- Ask about Crystal syntax, types, or patterns

The skill provides:
- Quick reference for Crystal syntax
- Best practices for code generation
- Common patterns and idioms
- Performance optimization tips
- Links to comprehensive documentation


## Example Usage

When you load this skill, you can ask:

- "How do I define a struct in Crystal?"
- "What's the difference between struct and class?"
- "How do I handle nil safely?"
- "Show me how to use macros"
- "How do I spawn a fiber?"

The skill will provide accurate, idiomatic Crystal code examples following best practices.

## Requirements

- Crystal compiler (for testing code examples)
- Node.js (for npx skills CLI)

## Credits

This skill is based on the [Crystal for Agents](https://gitlab.com/renich/crystal-for-agents) project by [Renich Nolet](https://gitlab.com/renich).

Original repository: https://gitlab.com/renich/crystal-for-agents

## Resources

- [Crystal Book](https://crystal-lang.org/reference/) - Official language documentation
- [Crystal Source](https://github.com/crystal-lang/crystal) - Reference implementation
- [Crystal Shards](https://crystal-lang.org/reference/shards/) - Package manager
- [skills.sh](https://skills.sh/) - Browse more skills
