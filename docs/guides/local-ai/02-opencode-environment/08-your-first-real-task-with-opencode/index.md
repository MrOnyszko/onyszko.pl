# 08 - Your First Real Task with OpenCode<no value>

In this post, I will show you how to give OpenCode its first meaningful task—something beyond toy examples that actually
moves the needle on your project.

## Why

Most tutorials stop at "Hello World" or adding a single function. That's not real work. The jump from tutorial examples
to actual projects is where people get stuck.

The second reason: small features are the sweet spot for OpenCode. They're focused enough for the agent to handle
reliably but meaningful enough that you actually ship them. Trying to have OpenCode build an entire feature set at once
leads to frustration on both sides.

---

## How

### Choosing the Right Task

**Criteria for a good first task:**

1. **Single responsibility** — One clear thing to do, not a bundle of features
2. **Self-contained** — Changes happen in one file or a small group of related files
3. **Measurable outcome** — You can see it working immediately after completion

Examples of good first tasks:

- Add input validation to an existing form field
- Refactor a long function into smaller, named helpers
- Extract a repeated code pattern into a reusable utility function
- Add error handling to a specific API endpoint
- Write unit tests for a small piece of logic that doesn't have coverage yet

**What to avoid:**

- Building entire new modules from scratch
- Redesigning complex UI components without detailed specifications
- Architectural changes across multiple services
- Tasks requiring external APIs or database migrations on the first try

> Start with something you could do yourself in 10-15 minutes. That's the right complexity level for OpenCode's first
> real task.

### The Prompt

**Writing an effective prompt:**

Structure your request like you're telling a junior developer what to do:

1. **What needs changing** — Specify files and functions
2. **Why it needs changing** — Explain the problem or goal
3. **How it should work** — Include examples if possible

Good prompt example:

```
The login form in @lib/features/auth/widgets/login_form.dart doesn't validate the password properly.
Currently it only checks if the field is empty.

Add password strength validation that:
1. Requires at least 8 characters
2. Requires at least one uppercase letter
3. Requires at least one number
4. Returns a localized error message using the intl package (follow the pattern in @lib/core/localization/)

Use Flutter's built-in FormFieldValidator<String?> pattern. The validator should return null if valid, or an error string if invalid.
```

This prompt:

- Points to specific files and functions
- Explains what's missing
- Specifies the exact behavior and output format
- References existing code to reuse

**Providing necessary context:**

If your project has conventions, mention them:

```
Follow the same pattern as other validators in @lib/features/auth/validators/.
Use FormFieldValidator<String?> for the function signature.
Return localized strings using AppLocalizations from @lib/core/localization/.
```

### Execution

**Step-by-step workflow:**

1. **Open OpenCode**

   ```bash
   cd /path/to/your/flutter/project
   opencode
   ```

2. **Make sure you're in Build mode (not Plan)**
    - Press Tab to check the mode indicator in the lower right
    - If it says "Plan", press Tab again to switch to "Build"

3. **Paste your prompt and hit Enter**

4. **Monitor progress**
    - Watch for file modifications in your editor/terminal
    - OpenCode shows which files it's reading and writing
    - OpenCode displays its thought process in real-time: files being read, code snippets being analyzed, changes being
      applied
    - If something looks wrong, interrupt with Esc

   If you see something unexpected, stop and ask yourself:
    - Did I provide enough context?
    - Is my prompt too vague?
    - Am I asking for too much at once?

5. **Test the result immediately**
   ```bash
   fvm flutter test
   ```

### Review and Refine

**Evaluating the output:**

1. **Read the changes OpenCode made**
    - Don't just accept them blindly
    - Check that it followed your instructions exactly

2. **Run tests**

   ```bash
   fvm flutter test
   ```

3. **Manual verification**
    - Test the specific functionality you asked for
    - Make sure nothing else broke

**Making corrections:**

If something's wrong, don't just say "fix it." Be specific:

```
The password validator should also require a special character (!@#$%^&*).
Also, the error messages should use the localization keys, not hardcoded strings.

Please fix these two issues.
```

Or use undo/redo:

```
/undo
# Now refine your prompt and try again
```

---

## Key Points Recap

1. **Start small** — Choose tasks that are focused enough to handle reliably but meaningful enough to ship.
2. **Be specific in prompts** — Point to files, explain why, include examples. Treat OpenCode like a junior developer
   who needs clear instructions.
3. **Monitor execution** — Watch what OpenCode is doing as it happens and intervene if something looks wrong.
4. **Test immediately** — Verify the output works before moving on to the next task.
5. **Use undo/redo for corrections** — Don't try to fix complex mistakes in place; revert and refine your prompt.
