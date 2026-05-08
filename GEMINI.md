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

### Available Tools
Here is a comprehensive list of available tools:

**Inspection & Context**
- **`get_electron_window_info`**: Get information about running Electron applications and their windows. Automatically detects any Electron app with remote debugging enabled (port 9222).
- **`list_electron_windows`**: List all available Electron window targets across all detected applications. Returns window IDs, titles, URLs, and ports. Use the returned IDs with any electron_* tool's targetId parameter to target specific windows.
- **`electron_take_screenshot`**: Take a screenshot of any running Electron application window. Returns base64 image data for AI analysis. No files created unless outputPath is specified. Pass `targetId` (from list_electron_windows) for unambiguous targeting when multiple Electron apps run on different debugging ports — `targetId` takes precedence over `windowTitle`.
- **`read_electron_logs`**: Read console logs and output from running Electron applications. Useful for debugging and monitoring app behavior.
- **`electron_get_title`**: Get the document.title of the focused Electron window. Read-only; safe in any security profile.
- **`electron_get_url`**: Get the window.location.href of the focused Electron window.
- **`electron_get_body_text`**: Get the first 500 chars of document.body.innerText (truncated for payload size).
- **`electron_find_elements`**: Analyze all interactive elements (buttons, inputs, selects, links) on the page with their properties, positions, and selectors. Returns JSON.
- **`electron_get_page_structure`**: Get an organized overview of page elements (buttons, inputs, selects, links) including detected framework. Returns JSON.
- **`electron_debug_elements`**: Get debugging info about top 10 visible buttons and form elements. Useful before clicking/filling.
- **`electron_verify_form_state`**: Inspect every `<form>` on the page: inputs, values, and HTML5 validity. Use after fill_input to confirm state.
- **`electron_query_text_by_selector`**: Read textContent of the first element matching the CSS selector. Returns "Element not found: <selector>" if no match.
- **`electron_query_attribute_by_selector`**: Read an HTML attribute (e.g., href, data-*, aria-*) via getAttribute. Returns "Element not found" or "Attribute not found" sentinels for missing cases.
- **`electron_query_value_by_selector`**: Read the .value of an input, textarea, or select. Returns "Element not found" or "Element has no value property" sentinels for missing/non-form cases.
- **`electron_query_visible_by_selector`**: Check whether an element is visually rendered (non-zero size, display !== none, visibility !== hidden, opacity > 0). Returns "true"/"false" or "Element not found".
- **`electron_query_enabled_by_selector`**: Check whether a form control is enabled (returns "true"/"false"). Sentinels: "Element not found", "Element has no disabled property" for non-form elements.

**Interactions**
- **`electron_click_by_selector`**: Click an element by CSS selector. Returns the element tag/text on success. Note: same-selector clicks within ~1s are rate-limited and return "Click prevented - too soon after previous click" — serialize calls (await each) to avoid this. Replaces the deprecated click_button — pass selector="button" to keep that behavior.
- **`electron_click_by_text`**: Click an element by visible text, aria-label, or title. Best for buttons/links. Returns confidence score on match. Note: same-element clicks within ~2s are rate-limited and return an error containing "Element click prevented - too soon after previous click" — serialize calls (await each) to avoid this.
- **`electron_double_click_by_selector`**: Double-click an element by CSS selector using CDP mouse events. Triggers handlers that synthetic dblclick misses (Monaco, canvas, etc.).
- **`electron_right_click_by_selector`**: Right-click an element by CSS selector using CDP mouse events. Triggers contextmenu and native context menu UI.
- **`electron_drag_from_to`**: Drag from a source element to a target element using CDP mouse events. Includes intermediate moves so drag libraries (react-dnd, dnd-kit) recognize the gesture.
- **`electron_hover_by_selector`**: Hover over element by CSS selector using CDP-level mouse events. Triggers tooltips/popovers that synthetic JS events miss (Radix UI, etc.).
- **`electron_hover_by_text`**: Hover over element by visible text using CDP-level mouse events. Triggers tooltips/popovers that synthetic JS events miss.
- **`electron_fill_input`**: Fill an input or textarea with a value. React-aware (uses native setter so controlled components update). Identify by selector OR placeholder/label.
- **`electron_select_option`**: Select a dropdown option by value or visible text. Identify the `<select>` via selector or adjacent label text.
- **`electron_navigate_to_hash`**: Navigate to a hash route (e.g., "#create"). Uses pushState + manual hashchange + popstate dispatch so both legacy listeners and react-router-dom v7 HashRouter pick it up.

**Waiting & Synchronization**
- **`electron_wait_for_selector`**: Wait for a CSS selector to match. MutationObserver-based; returns "Found" or "Timeout" within timeoutMs (default 5000ms).
- **`electron_wait_for_text`**: Wait until a substring appears in document.body. MutationObserver-based; returns "Found" or "Timeout" within timeoutMs (default 5000ms).
- **`electron_wait_for_navigation`**: Wait for a URL change (or for the URL to contain expectedUrlSubstring). Default 10000ms, capped at 120000ms.
- **`electron_wait_for_function`**: Poll a JS expression until it returns truthy. operationType=eval (validated). Default 5000ms, polled every 100ms.
- **`electron_wait_for_load_state`**: Wait for page lifecycle: load | domcontentloaded | networkidle. Default 30000ms, networkidle = 500ms quiet window.

**View & Scroll**
- **`electron_scroll_to_element`**: Scroll an element into view via scrollIntoView. Defaults to block:center so sticky headers do not obscure it.
- **`electron_scroll_by_pixels`**: Scroll the window by a pixel delta. Returns the new (scrollX, scrollY) so callers can verify movement.
- **`electron_get_viewport_size`**: Read viewport innerWidth/innerHeight and devicePixelRatio. Returns JSON {width, height, devicePixelRatio}.
- **`electron_get_scroll_position`**: Read scroll metrics for the window (default) or an element matching `selector`. Returns JSON {scrollX, scrollY, maxScrollX, maxScrollY}; element variants source the values from el.scrollLeft / scrollTop / scrollWidth - clientWidth / scrollHeight - clientHeight.

**Keyboard Events**
- **`electron_send_keyboard_shortcut`**: Dispatch a keyboard shortcut (e.g., "Ctrl+N", "Enter") on document. Best for app-level hotkeys, not input typing.
- **`electron_press_key`**: Press a single key with optional modifiers via CDP. Use this for real keyboard input (cursor moves, IME) — for app hotkeys prefer electron_send_keyboard_shortcut. Note: macOS OS-level shortcuts (Cmd+A, Cmd+C/V/X, Cmd+Z) are bypassed by CDP synthetic events; use electron_eval (.select() / navigator.clipboard.*) instead.

**Storage Management**
- **`electron_local_storage_get_item`**: Read a single key from localStorage. Returns the raw string or "Item not found: <key>". Returns "Storage unavailable" when the renderer blocks access.
- **`electron_local_storage_set_item`**: Write a single key/value to localStorage. Value is stored verbatim; pre-serialize objects with JSON.stringify.
- **`electron_session_storage_get_item`**: Read a single key from sessionStorage. Returns the raw string or "Item not found: <key>".
- **`electron_session_storage_set_item`**: Write a single key/value to sessionStorage. Value is stored verbatim; cleared on window close (unlike localStorage).
- **`electron_clear_storage`**: Clear local/session/cookie storage scopes. Cookies are best-effort (HttpOnly survives) — for full wipe use session.clearStorageData in main process.

**Execution & Logging**
- **`electron_eval`**: Execute custom JavaScript with structured error reporting. Last resort — use specific tools (click, fill, query) when possible.
- **`electron_console_log`**: Emit a message via console.log in the renderer. Useful for test sanity checks.

### Troubleshooting
- If multiple Electron apps/windows are running, use `list_electron_windows` to find the correct `targetId` and pass it to your tool calls.
- If a click fails due to "Click prevented - too soon after previous click", serialize calls and wait between them.
- If you need to press special keys, use `electron_press_key` or `electron_send_keyboard_shortcut`.
