# AGENTS.md — plugin-ntfy

> **Start at the roadmap and board on [lemonfiber.app](https://lemonfiber.app),
> rendered from the report of where every unreleased version stands. Then the
> rules** every repository shares:
> [working in the repositories](https://github.com/lemonfiber/spec/blob/main/50-governance/working-in-the-repositories.md)
> and [the rules for agents](https://github.com/lemonfiber/spec/blob/main/50-governance/ai-contributors.md).
> This file holds only what is true of this repository.

## What this repo is

A first-party plugin the bundle offers and does not ship: ntfy, the stack's own
push server (`B9-R11`). The
[bundle contract](https://github.com/lemonfiber/spec/blob/main/20-architecture/contracts/bundle.md)
says how the catalogue pins it as an `[[offered]]` and how lemonfiber carries it
(`ARCH-R394`).

## The rules you cannot break

- **The core speaks ntfy itself** (`ARCH-R399`). Delivery, proving and the
  account lemonfiber publishes as belong to the core. This repository runs the
  server and nothing else: no adapter, no image, no code.
- **One service, on the upstream image pinned by digest**, started with
  `args = ["serve"]` because the image runs nothing until told to (`ARCH-R407`).
- **Deny-all, one account, one prefix.** `config/ntfy/server.yml` declares the
  `lemonfiber` user with `{{secret:password|bcrypt}}`, the bcrypt hash of the minted
  secret `ntfy:password` (`ARCH-R406`), and grants it read and write on
  `lemonfiber-*` alone. The core reads that secret to publish and to read a topic's
  history (`ARCH-R402`). The account is never an admin, and the `account` job in
  `plugin.yml` holds it to exactly that.
- **`bind = "proxied"`** (`ARCH-R408`). Its port is published on loopback alone for
  lemonfiber, and the household reaches it only through the proxy's TLS, as `ntfy`.
- **Every file it writes is under `/config`.** It runs `confined`, as the
  operator's user, with a read-only root.
- **It fills no capability and replaces no part.** It sits in the bundle's
  `notify` profile, which runs beside whichever forms run, so `[plugin].forms` names
  every form the bundle declares, `proxy` included (`ARCH-R410`). No service
  declares `replaces`.
- **Every service declares `reaches` and `asks_for`** (`ARCH-R270`).
- **No Python here.** The reader is spec's `first-party-plugin` workflow's.
