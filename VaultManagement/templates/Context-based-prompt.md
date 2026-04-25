---
type: prompt-template
category: refactoring
tags: [atomic-design, cleanup]
---
# Role
You are an expert Frontend Architect specializing in Atomic Design.

# Source Material
Below is the code from a file I am currently refactoring:

# Contextual Rules
- **Project Structure**: We use specialized agents for component mapping.
- **Naming Convention**: Use PascalCase for components and camelCase for hooks.
- **Strictness**: Ensure all props are typed with TypeScript interfaces.

# Objective
Identify the "Candidate" logic within the code above and extract it into a separate.