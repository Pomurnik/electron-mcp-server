# Electron MCP Server - Gemini Context

This extension provides an MCP server for interacting with and automating Electron applications via Chrome DevTools Protocol.

## Security Level
The server runs by default at `SECURITY_LEVEL=balanced`, which allows UI interactions, DOM queries, and property access, but blocks dangerous operations like assignment or raw function calls (unless they are safe UI functions).

## Usage Guide
The Electron MCP server exposes tools to inspect and interact with Electron applications. Most tools take top-level arguments such as `selector`, `text`, or `value`, as well as optional window targeting arguments (`targetId` or `windowTitle`).

### Recommended Workflow
1. **Inspect**: Use `electron_get_page_structure` or `electron_find_elements` to analyze interactive elements on the page.
2. **Target**: Identify elements using specific selectors or text content.
3. **Interact**: Use commands like `electron_click_by_text`, `electron_click_by_selector`, or `electron_fill_input`.
4. **Verify**: Check state changes with `electron_take_screenshot` or querying elements.

### Key Tools
- **`electron_get_page_structure`**: Get an organized overview of page elements.
- **`electron_find_elements`**: Analyze all interactive elements on the page.
- **`electron_click_by_text`**: Click elements by visible text, aria-label, or title (highly recommended for reliability).
- **`electron_click_by_selector`**: Click an element by its CSS selector.
- **`electron_fill_input`**: Fill an input field using `placeholder` (or label text) or `selector` with a `value`.
- **`electron_select_option`**: Select dropdown options by `value` or visible text.
- **`electron_take_screenshot`**: Take a screenshot of the running Electron app. Optional `targetId` helps target specific windows.
- **`electron_eval`**: Execute custom JavaScript with structured error reporting (use as a last resort).

### Troubleshooting
- If multiple Electron apps/windows are running, use `list_electron_windows` to find the correct `targetId` and pass it to your tool calls.
- If a click fails due to "Click prevented - too soon after previous click", serialize calls and wait between them.
- If you need to press special keys, use `electron_press_key` or `electron_send_keyboard_shortcut`.
