# 06 - How OpenCode Actually Works<no value>

In this post, I will explain how OpenCode processes your requests as an agentic coding assistant and what you should
realistically expect from it.

## Why

OpenCode is not a chatbot that answers questions. It's an agent that actively works on your codebase—reading files,
making changes, running commands, and iterating based on feedback. Understanding this fundamental difference prevents
frustration and helps you use OpenCode effectively.

The second reason: OpenCode's agentic loop has specific strengths and limitations. Knowing what it can and cannot do
saves you time. You'll stop trying to make it do things it wasn't designed for and instead leverage its real
capabilities.

---

## How

### The Agentic Loop

OpenCode follows a decision-making flow when you ask it to work on your project:

1. **Interpret** — It reads your prompt and figures out what you want done.
2. **Plan** — In Plan mode, it proposes how to implement the feature or fix without making changes yet.
3. **Execute** — In Build mode, it actually modifies files, runs commands, and applies changes.
4. **Iterate** — If something goes wrong or you request adjustments, it revises its approach.

Switch between modes using Tab in the TUI. Plan mode is for complex features where you want to review the approach
first. Build mode is for straightforward changes or when you trust the plan.

> OpenCode doesn't "chat" with you—it works alongside you. Think of it as a junior developer who needs clear
> instructions and context, not small talk.

### What OpenCode Can Do

**1. Read and understand your codebase**

OpenCode scans your project structure and uses LSP servers to understand code semantics. You can ask:

```
How is authentication handled in @lib/services/auth_service.dart?
```

The `@` key triggers file fuzzy-search, letting you reference specific files.

**2. Add features with a plan**

Describe what you want, switch to Plan mode, review the proposed approach, then switch back to Build mode to execute:

```
Add a "favorites" feature to the notes app. When a user long-presses
a note in @lib/screens/notes_list_screen.dart, toggle its favorite
status. Add a filter in the NotesBloc to show only favorited notes.
Show a filled star icon for favorited notes in @lib/widgets/note_card.dart
```

**3. Make direct changes**

For simple tasks, skip the plan and go straight to execution:

```
Add validation to the email TextField in @lib/screens/login_screen.dart.
Validate that the input matches email regex and show an error message
below the field if invalid. Use the same validation pattern as in
@lib/text/validators.dart
```

**4. Undo and redo changes**

If OpenCode goes off track, use `/undo` to revert the last change and try again:

```
/undo
```

You can also `/redo` to restore a change you just undid.

### What OpenCode Can't Do

**1. Magic without context**

OpenCode isn't psychic. It needs context about your project. The more specific your instructions, the better it
performs. Vague prompts like "make this better" yield poor results.

**2. Design from scratch**

If you need a complete UI redesign or architectural overhaul, OpenCode helps implement but doesn't invent. Provide
design references (drag UI mockups into the terminal) or detailed widget specifications.

**3. Debug complex runtime issues alone**

OpenCode can fix syntax errors and logic mistakes it can infer from code. For subtle race conditions or integration
issues, you'll still need to test manually and provide feedback.

### Setting Expectations

**Best practices for collaboration:**

1. **Be specific about what you want**
    - Good: "Add validation for the email field in the user registration form in @lib/screens/register_screen.dart"
    - Bad: "Fix the registration form"

2. **Provide examples when possible**
    - Include before/after code snippets from your Flutter project
    - Share screenshots of desired UI behavior (drag images into the terminal)

3. **Use Plan mode for complex features**
    - Review the approach first
    - Give feedback on the plan before letting it execute

4. **Iterate with undo/redo**
    - If the output isn't right, use `/undo` and refine your prompt
    - OpenCode learns from your corrections

**Common mistakes to avoid:**

- Asking too many things at once — break tasks into smaller requests
- Not providing enough context about your project structure
- Forgetting to specify which files or functions to modify
- Expecting it to know business logic not written in the codebase

---

## Key Points Recap

1. **OpenCode is an agent, not a chatbot** — It actively reads and modifies your codebase rather than just answering
   questions.
2. **Plan mode vs Build mode** — Use Plan mode for complex features where you want to review the approach first; Build
   mode for straightforward changes.
3. **Context matters** — The more specific your instructions and the better your project context (via AGENTS.md), the
   better OpenCode performs.
4. **Undo/redo is your safety net** — Use `/undo` when OpenCode goes off track, then refine your prompt.
5. **Break complex tasks into smaller ones** — OpenCode works best on focused, single-purpose requests rather than
   sprawling feature requests.
