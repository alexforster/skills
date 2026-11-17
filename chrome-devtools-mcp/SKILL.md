---
name: chrome-devtools-mcp
description: Use Chrome DevTools MCP when a task needs browser interaction, live page inspection, rendered styling, console or network evidence, or performance and accessibility checks. Skip it for read-only fetches that do not need a full browser.
---

# Chrome DevTools MCP

Use the Chrome DevTools MCP when the answer depends on what a page does or displays in a running Chrome session. Typical uses include reproducing a browser bug, checking a rendered UI, interacting with a page, diagnosing console or network failures, inspecting computed styles, and measuring browser performance or accessibility.

For code-only changes, use the repository and its normal checks unless runtime browser behavior is part of the task. For broad integration-test requests, use the project's designated integration-test workflow; a DevTools session is useful for focused browser reproduction and inspection, not a substitute for a test suite.

## Choose the relevant tools

* Use `list_pages` to identify open tabs and their `pageId`s. If the user refers to an existing page, inspect that page before navigating elsewhere. Use `new_page` when a separate tab is useful; use `navigate_page` when changing the selected tab is intended.
* Use `take_snapshot` to inspect the page's accessible structure and obtain current element `uid`s. Use `wait_for` when there is a specific expected text to wait for.
* Use `click`, `fill`, `press_key`, or `type_text` for page interactions. Take a fresh snapshot after navigation or a substantial page update; element `uid`s can become stale.
* Use `take_screenshot` when visual layout or a visual comparison matters. Use `get_css_styles` to understand matched styles and the cascade for a specific element.
* Use `list_console_messages` to find browser errors and `get_console_message` for details about a relevant message.
* Use `list_network_requests` to locate relevant requests and `get_network_request` to inspect a selected request or response when needed.
* Use `lighthouse_audit` for an accessibility, SEO, or best-practices audit. Use `performance_start_trace`, `performance_stop_trace`, and `performance_analyze_insight` when measuring or diagnosing runtime performance.
* Use `evaluate_script` for a small, focused read of runtime page state when snapshots and dedicated inspection tools do not expose the needed detail. Prefer normal UI interactions for actions.

## Work from the page state

First confirm the target page and inspect its current state. Navigate only when the requested task calls for a URL or route change. Wait for a concrete loading condition when one is known, then inspect the updated page before interacting. After an interaction that matters to the task, verify the visible result with a fresh snapshot or other relevant observation.

Keep inspection narrow. Retrieve only the console messages, network requests, DOM data, styles, or performance details needed to answer the question. Network request inspection can expose cookies, authorization headers, and private response data; avoid retrieving or repeating those values unless the task requires them.

Treat inspection as read-only unless the user asked for an action. Submitting forms, changing account settings, deleting or publishing content, purchasing, and other consequential page actions require a clear user instruction. Do not use page scripts to bypass the UI's access controls or to retrieve unrelated hidden data.

When reporting results, distinguish browser observations from conclusions inferred from them. Include the page or interaction context needed to make the evidence understandable, and summarize sensitive values rather than copying them.
