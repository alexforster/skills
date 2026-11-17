---
name: computer-use
description: Use when interacting with native macOS GUI applications through shell-based window management, input injection, or screen capture, especially when no dedicated computer-use tool is available.
---

# macOS screen interaction from a shell

Use this technique for a native GUI app when no dedicated computer-use tool is available. The technique is for macOS. It does not guarantee that every app exposes the same accessibility controls or accepts synthetic input. Keep window management, input injection, and visual verification as separate steps.

## Launch and identify the target

If you can launch an executable directly, preserve its output and PID:

```sh
APP=/path/to/application
PROCESS_NAME='Application Process Name'
LOG=$(mktemp /tmp/gui-app.XXXXXX.log)
"$APP" >"$LOG" 2>&1 &
PID=$!
sleep 2
```

For a packaged `.app`, use `open -a 'Application Name'` instead. This command can focus or reuse an app that is already running. It does not necessarily create a new window.

Inspect the accessible windows. If necessary, create a window through the app's menu or an exposed accessibility action. The application name, executable name, and process name can differ. Use the process name that Activity Monitor shows, or query System Events. Keep the app running until the interaction and capture are complete.

## Focus, inspect, move, and resize

System Events can address an application process by name. It can also manipulate accessible windows:

```sh
osascript \
  -e "tell application \"System Events\" to tell process \"$PROCESS_NAME\" to set frontmost to true" \
  -e "tell application \"System Events\" to tell process \"$PROCESS_NAME\" to set position of window 1 to {300, 100}" \
  -e "tell application \"System Events\" to tell process \"$PROCESS_NAME\" to set size of window 1 to {900, 632}"
```

Inspect accessible windows before you assume their order or geometry:

```sh
osascript -e "tell application \"System Events\" to tell process \"$PROCESS_NAME\" to return {name of every window, position of window 1, size of window 1}"
```

Window index 1 is an example. If the app has several windows, select the intended window. The reported frame size can include decorations such as the title bar.

Resize the window, then query it again. Use the actual client/content size of the app. Do not assume that the requested frame size is the drawable size. In the tested setup, a 900×632 frame yielded a 900×600 client area.

If an app exposes accessibility elements, `osascript` can also click them by role/name. A coordinate `click at {x, y}` returned success. It did not deliver a mouse event to the tested native app. A successful AppleScript command does not prove that the target received input.

## Verify each action

A zero exit status proves only that the automation command was accepted. After each important transition, verify the intended app, window, focused control, and visible state.

In browser workflows, inspect the address-bar value and resulting URL after navigation. Autocomplete can replace the text with an old suggestion. If Enter does not submit a search, inspect the visible suggestions or submit control. Activate the intended suggestion or control. Confirm the result by URL/title and visible page content. Do not assume that the typed text was submitted.

If an AppleScript query fails with an invalid target or type, inspect the accessible children and their roles/values. Do this before you guess a deeper reference. Menus and popups can be separate elements from the main window.

One Chrome-to-Bing search test needed these checks. `open -a` reused an existing Chrome process. Autocomplete rewrote the typed destination. Enter did not submit the search, so the visible suggestion was activated. The URL, title, and visible content confirmed the result. This is a diagnostic example, not a Chrome-specific recipe.

## Inject mouse, keyboard, and wheel events

Quartz `CGEvent` can post low-level input. The coordinates below are global desktop coordinates. Replace the example point with a coordinate derived from the current window and target UI. In the tested setup, a mouse-move event was necessary before the button events. Without it, the button events used a stale pointer position.

```sh
swift -e '
import CoreGraphics
import Foundation
let source = CGEventSource(stateID: .hidSystemState)
let point = CGPoint(x: 500, y: 400) // replace with the target screen point
for type in [CGEventType.mouseMoved, CGEventType.leftMouseDown, CGEventType.leftMouseUp] {
    if let event = CGEvent(mouseEventSource: source,
                           mouseType: type,
                           mouseCursorPosition: point,
                           mouseButton: .left) {
        event.post(tap: .cghidEventTap)
    }
    usleep(100_000)
}
'
```

For a right-button drag, post these events in order:

1. Post `mouseMoved` to the start point.
2. Post `rightMouseDown` at the start point.
3. Post `rightMouseDragged` at the destination.
4. Post `rightMouseUp` at the destination.

Use `.right` for the mouse button. Small delays between events let the app process them.

Keyboard events use macOS virtual key codes. Map the key for the target app/keyboard layout. Do not assume that codes are portable:

```swift
if let event = CGEvent(keyboardEventSource: source,
                       virtualKey: keyCode,
                       keyDown: true) {
    event.post(tap: .cghidEventTap)
}
```

To post a vertical scroll event, use `CGEvent(scrollWheelEvent2Source:units:wheelCount:wheel1:wheel2:wheel3:)`. The sign and meaning of the scroll value depend on the app/platform behavior. Observe the result instead of assuming a direction.

## High-DPI coordinate model

Do not treat “screen coordinates,” “window coordinates,” “drawable pixels,” and “app logical coordinates” as interchangeable. A typical path is:

```text
global screen point → window/client point → drawable/backing pixel → app logical canvas
```

The coordinate space of the input API can differ from the backing-buffer space of the renderer. On the tested macOS display, System Events window geometry and `CGEvent` positions aligned in desktop coordinates. A high-density window had a backing buffer twice the client width and height. A point multiplied by two missed the intended window location.

**Do not multiply a screen-space click by the Retina/backing scale unless the receiving API explicitly expects backing pixels.** Let the target framework convert native window events to its render/logical coordinates. Alternatively, perform one explicit conversion at the adapter boundary. Do not do both.

The backing scale is not a universal constant. It can depend on the window's display, display mode, and whether the app requested a high-density surface. If you move a window between displays, its effective scale can change. If you resize a window, its drawable dimensions change.

Query the actual window/client and drawable/backing sizes using the target framework or windowing API. If the app exposes a logical canvas, use its dimensions and the framework's presentation transform for hit testing. Validate the conversion with an actual UI event. Confirm that the intended control/state changed. Avoid hard-coded scale factors. Do not extrapolate one window's measurements to another.

## Capture the target window

`screencapture -l` captures a specific on-screen window ID. Obtain that ID from Core Graphics. Filter for the actual process name:

```sh
WINDOWS=$(swift -e '
import CoreGraphics
import Foundation
let processName = "Application Process Name" // replace
let windows = CGWindowListCopyWindowInfo(.optionOnScreenOnly, kCGNullWindowID) as! [[String: Any]]
for window in windows where (window[kCGWindowOwnerName as String] as? String) == processName {
    let id = window[kCGWindowNumber as String] ?? "?"
    let layer = window[kCGWindowLayer as String] ?? "?"
    let bounds = window[kCGWindowBounds as String] ?? "?"
    let title = window[kCGWindowName as String] ?? ""
    print("id=\(id) layer=\(layer) bounds=\(bounds) title=\(title)")
}
')
printf '%s\n' "$WINDOWS"
# Choose one ID matching the intended window's bounds; do not use the full list.
WINDOW_ID=12345 # replace with one reported ID
screencapture -x -l "$WINDOW_ID" /tmp/application-window.png
```

Capture only the target to avoid including unrelated windows. System Events automation may require Accessibility permission. Screen capture may require Screen Recording permission. Grant only the permissions needed for the intended test.

## Cleanup and limits

Terminate only the process launched for the test. For example, use `kill -TERM "$PID"`, then `wait "$PID"`. Restore any temporary app configuration. Rebuild before you use the normal executable.

Synthetic input proves only the path exercised on that machine. It does not establish behavior on other platforms, display scales, or accessibility configurations.
