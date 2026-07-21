---
name: comments-with-references
description: Use when adding or revising an inline code comment for a non-obvious implementation decision, workaround, compatibility constraint, or bug fix that needs a nearby authoritative URL.
---

# Comments With References

Use a short, nearby URL to preserve the evidence behind a non-obvious implementation decision. The comment must explain why the code exists; the link must let a future reader verify that reason.

## Add a Reference When

Add a reference for an external constraint, framework limitation, compatibility workaround, security decision, protocol quirk, or issue that makes an otherwise simpler implementation incorrect.

Use the most direct stable source: official documentation, a specification, an upstream issue, or the relevant project decision record. Link to the exact page, issue, or section—not a search page or a home page.

```python
# See: https://example.com/relevant-docs
# NOTE: Problem described in: https://example.com/issue
result = retry_transient_failure(operation)
```

Keep the reference adjacent to the code it explains. Update or remove it when the constraint or implementation changes.

## Do Not Add a Reference When

Do not comment code that is already clear from names, types, and control flow. Improve the code instead of explaining an obvious assignment, branch, loop, or function call.

Do not use a URL as a substitute for the explanation. State the relevant constraint in the comment, then cite the source when the reader needs evidence or deeper context.

```python
# Bad: documents an obvious operation.
total += item.price

# Good: preserve the reason for an unusual fallback.
# The provider returns an empty page while its cursor is still valid.
# See: https://example.com/api-pagination
if not page.items and page.next_cursor is not None:
    return fetch_page(page.next_cursor)
```

## Review Checklist

- Does the comment explain a non-obvious constraint or decision?
- Does the URL directly support that explanation?
- Is the source authoritative and stable?
- Is the comment close to the affected code?
- Would clearer code eliminate the need for the comment?
