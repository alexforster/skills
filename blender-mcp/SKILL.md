---
name: blender-mcp
description: Use Blender MCP when a task requires inspecting or changing an open Blender scene, querying Blender data, running bpy code, or rendering through the connected Blender session.
---

# Blender MCP

Use the connected Blender MCP tools for live Blender scene work. The Blender MCP add-on must be enabled in an open Blender instance and its MCP server must be running.

Inspect the current scene before making changes. Prefer the dedicated Blender summary, detail, screenshot, and render tools; use `execute_blender_code` for operations that need Blender's Python API. Verify mutations by inspecting the scene or querying the affected object afterward.
