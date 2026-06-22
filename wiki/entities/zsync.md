---
type: entity
title: "zsync"
created: 2026-06-21
updated: 2026-06-21
tags:
  - async
  - runtime
  - zig
status: seed
entity_type: repository
language: Zig
repo_status: active
purpose: "Async runtime for Zig"
related:
  - "[[Zig]]"
  - "[[zquic]]"
  - "[[GhostKellz]]"
---

# zsync

An async runtime for [[Zig]]. By [[GhostKellz]].

It provides the asynchronous execution foundation that the no-external-deps Zig
networking stack relies on, complementing [[zquic]] (QUIC transport) and
[[zcrypto]] (crypto). It also appears as an integration module inside [[zqlite]].

Built on [[Zig]].
