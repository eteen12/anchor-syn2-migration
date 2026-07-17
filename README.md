# Anchor: syn 2.0 migration — merged OSS contribution

[![PR](https://img.shields.io/badge/PR-%234523-blue?logo=github)](https://github.com/otter-sec/anchor/pull/4523)
[![Status](https://img.shields.io/badge/status-merged-8250df?logo=github)](https://github.com/otter-sec/anchor/pull/4523)
[![Fixes](https://img.shields.io/badge/fixes-%234521-green?logo=github)](https://github.com/otter-sec/anchor/issues/4521)

This repo showcases a contribution I made to [otter-sec/anchor](https://github.com/otter-sec/anchor)
(a fork of [Anchor](https://github.com/coral-xyz/anchor), the Solana smart-contract framework):

> **[fix(lang/syn): migrate anchor-syn and proc-macro crates to syn 2.0 (#4523)](https://github.com/otter-sec/anchor/pull/4523)**
> Merged 2026-05-14 · **+328 / −203** across **30 files**

The full diff is in [`pr-4523.patch`](./pr-4523.patch).

## The problem

Anchor's IDL generation (`anchor-syn`) was built on `syn` 1.x, which cannot parse
modern Rust syntax such as `impl Clone + use<T>` precise-capture bounds (Rust 1.82+).
When `CrateContext::parse` hit syntax it couldn't parse, it **silently produced an
incomplete IDL** — programs compiled fine but shipped broken interface definitions
([#4521](https://github.com/otter-sec/anchor/issues/4521)).

## What I did

Upgraded `syn` from 1.x to 2.0 across `anchor-syn` and every dependent proc-macro crate:

- `lang/attribute/{program,account,access-control,error,event,constant}`
- `lang/derive/{accounts,serde,space}`

### Key syn 2.0 breaking changes addressed

| syn 1.x | syn 2.x |
|---------|---------|
| `attr.path` (field) | `attr.path()` (method) |
| `attr.tokens` (field) | `attr.meta` / `MetaList::tokens` |
| `NestedMeta` | `syn::Meta` |
| `parse_meta()` | `attr.meta` |
| `LifetimeDef` | `LifetimeParam` |
| `Expr::Type` | `FnArg::Typed` |
| `ParseMacroInput` | `Parse` |
| `parse_terminated(f)` | `parse_terminated(f, Token![,])` |

### Removed the silent-fail bandaid

`CrateContext::parse` now hard-fails with a clear error when it cannot parse the
crate, instead of silently emitting an incomplete IDL. This was only safe to do
because syn 2.0 can parse the modern syntax that syn 1.x choked on.

### Regression test

Added an IDL regression test using `+ use<T>` precise-capture syntax — the exact
pattern that previously triggered the silent-fail path. It now compiles and
generates correct IDL.

## Commits

- `b049ad8` fix(lang/syn): migrate anchor-syn and proc-macro crates to syn 2.0
- `3ad7ccc` Update CHANGELOG.md
- `d196c84` test(idl): integrate `+ use<T>` regression into generics test program

## Links

- Merged PR: https://github.com/otter-sec/anchor/pull/4523
- Issue it fixes: https://github.com/otter-sec/anchor/issues/4521
- My fork with the branch: https://github.com/eteen12/anchor/tree/fix/syn2-crate-context-parse

---

*The patch here is my own work, contributed to Anchor under its Apache-2.0 license.*
