# Browser Automation Setup

Standard sequence for skills that use Claude in Chrome MCP tools to fetch web pages.

## Tab Setup

```
1. tabs_context_mcp → get browser state (confirms Chrome + extension are active)
2. tabs_create_mcp → create a new tab in the MCP group
3. navigate(url, tabId) → go to the target URL
4. Extract content (see below)
```

## Content Extraction (choose the right tool)

**For simple pages** (single job posting, article, form):
- `get_page_text` — returns plain text. Safe when the page has one main content area.

**For complex/dynamic pages** (job boards, search results, listing pages, dashboards):
- `javascript_tool` with a targeted selector — extract only the data you need
- `read_page(filter="interactive")` — get interactive element refs for form filling
- `find("search bar")` or `find("submit button")` — locate elements by description
- **Do NOT use `get_page_text`** — it returns the entire page and can blow out the context window, making the conversation unrecoverable.

**For form filling** (applications, sign-in pages):
- `read_page(filter="interactive")` — get all input fields with refs
- `find` — locate elements by natural language description (useful for buttons, radio groups)
- `form_input(ref, value)` — set values on input fields
- `computer(action="left_click", coordinate=[x,y])` — click elements that `form_input` can't reach (radio buttons, custom dropdowns)

## Context Window Safety

**Rule of thumb**: If a page has navigation, sidebars, related listings, or footer content, don't use `get_page_text`. Use `javascript_tool` or `read_page` to extract only what you need.

If you're unsure, take a `screenshot` first to see what the page looks like, then decide on extraction strategy.

## Error Handling

- If `tabs_context_mcp` returns no tabs or an error → ask the user to confirm Chrome is open with the Claude in Chrome extension active.
- If `navigate` fails or the page doesn't load → ask the user to paste the content directly.
- If `get_page_text` returns empty or unusable content → try `read_page` as a fallback, then ask the user to paste if that also fails.
- If the page requires authentication → tell the user to sign in, then say "continue" when ready. Never create accounts or enter passwords.
- Do not retry a failing page more than once. Move on and ask the user for the content.

## ATS-Specific Patterns

For job application forms (Greenhouse, Lever, Workday), see `shared/references/ats-patterns.md` for detailed navigation and interaction patterns.
