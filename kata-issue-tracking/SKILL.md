---
name: kata-issue-tracking
description: Compatibility entry for retired Kata workflows; directs issue work to Forgejo.
version: 2.0.0
---

# Forgejo replaced Kata

Kata is retired. Use Forgejo issues for work tracking. Do not start Kata, its
bridge or federation, or use Beads/DoltHub, Linear or GitHub Issues as another
tracker. Retain historical exports as offline recovery archives.

Use the configured Forgejo MCP issue tools to search, create, assign, comment,
label and close issues. If the project supplies `monocloud_issue_tracker`, use
its configured backend and legacy-reference resolver. Do not infer migrated
issue numbers from titles or call a retired service for old IDs.

Search before creating. Put acceptance criteria and stable decisions in the
issue body, progress in comments, and blockers in native dependencies. Assign
before implementation; close with completion evidence and reread the resulting
state. Keep unfinished work open and release its assignment at handoff.

This historical skill name remains only so existing installations get the
retirement correction when their managed skills source refreshes.
