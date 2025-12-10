# Spring Boot Backend MSA Template

## File Structure

```
.claude/
└── skills/
    └── spring-boot-backend-msa/
        ├── SKILL.md                        # Backend code generation guidelines (main entry point)
        └── references/
            ├── generation.md               # Basic rules and procedures for code generation (highest priority)
            ├── domain-events.md            # Domain event publishing and handling rules
            ├── eventstorming.md            # Event Storming-based design rules
            ├── fixed-generation.md         # Fixed pattern code generation rules
            ├── package-structure.md        # Backend project package structure
            ├── technical-stack.md          # Backend technology stack and framework guide
            └── test-generation.md          # Backend test code generation rules
--
PRD.txt                                     # Project requirements definition template
```

## Usage

### 1. Open Terminal
Open a terminal in your working directory.

### 2. Set Node Version (Node 20+)
```bash
nvm use 22
```

### 3. Run Claude Code Command
Execute Claude Code in your IDE.

### 4. Add PRD File to Prompt
Add `@PRD.txt` to the prompt area and execute.