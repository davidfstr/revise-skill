# Blank line between adjacent strongly related blocks

**Trigger:** A blank line separating two code blocks that handle closely related concerns in the same logical flow (e.g., two sequential error checks on the result of the same operation).

**When to revise:** When the blocks are tightly coupled steps -- e.g., checking `FileNotFoundError` then checking `returncode`, both handling failures of the same `subprocess.run()` call.

**When NOT to revise:** When the blocks handle genuinely different phases of the function, or when a blank line aids readability by separating a long block from unrelated logic below.

**Fix:** Remove the blank line to visually group the related blocks.
