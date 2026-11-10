# ntfy for lemonfiber

**The household's own push server: alerts reach your phones through this house, not through a company's service.**
ntfy is the self-hosted push channel lemonfiber offers first for notifications.
Without it: alerts reach a phone only through a push service you set up and run yourself.

## How lemonfiber carries it

The default bundle in
[lemonfiber-plugins](https://github.com/lemonfiber/lemonfiber-plugins) pins this
repository as an offered plugin, by a revision and the digests of its manifest
and its tree, and lemonfiber compiles that tree in. The stack does not run it
until the operator installs it. lemonfiber then installs it offline from that
tree, in whichever form the stack runs.

Its image is ntfy's own, `binwiederhier/ntfy`, pinned by digest and started
with `serve`. This repository builds no image, so the bundle pins its revision.

lemonfiber speaks ntfy itself. It publishes each alert to a topic and proves a
delivery by reading the topic's history back. All this plugin does is run the
server those messages go through.

## How the server is set up

lemonfiber writes `config/ntfy/server.yml` into ntfy's configuration directory
before ntfy first starts.

| Setting | Value | Why |
| --- | --- | --- |
| `auth-default-access` | `deny-all` | Nobody reads or publishes to a topic without an account |
| `auth-users` | `lemonfiber`, an admin | The account lemonfiber publishes and reads history as. Its password is minted at install as the secret `ntfy:password` and written here as the bcrypt hash ntfy keeps |
| `enable-signup` | `false` | Nobody makes an account through the web app |
| `enable-login`, `require-login` | `true` | The web app asks for an account before it shows anything |
| `cache-file` | `/config/cache.db` | Messages are kept on disk and survive a restart, so a topic's history can be read back to prove a delivery |
| `auth-file` | `/config/auth.db` | Accounts and access rules |
| `attachment-cache-dir` | `/config/attachments` | Files sent with a message |
| `listen-http` | `:2586` | The port it answers on, published on the household network |

Two settings come from lemonfiber's settings, and the operator can change them:

| Setting | Variable | Default | What it is |
| --- | --- | --- | --- |
| `ntfy:base-url` | `NTFY_BASE_URL` | `http://localhost:2586` | The address phones reach ntfy at, used in the links it writes for attachments. Set it to the address the household uses, such as `https://ntfy.<domain>` behind the proxy |
| `ntfy:cache-duration` | `NTFY_CACHE_DURATION` | `72h` | How long a message stays in a topic's history. A delivery can be proved, and a phone that was offline can catch up, only within this window |

The service runs `confined`, as the operator's user, with a read-only root. Every
file it writes is under its configuration directory. It sends nothing beyond this
machine.

## What is here

| Path | What it is |
| --- | --- |
| `plugin.toml` | The plugin's manifest |
| `config/ntfy/server.yml` | Its starting configuration |
| `recordings/ntfy/` | The responses its proofs are held to |
| `targets.toml` | The lemonfiber its checks run against |
| `proofs.json` | What those checks said, committed |

## Checks

The plugin verdict comes from spec's
[`first-party-plugin`](https://github.com/lemonfiber/spec/blob/main/.github/workflows/first-party-plugin.yml)
workflow. Its `plugin` job reads the manifest as the lemonfiber in `targets.toml`
reads it, checks every proof against its recording, and checks every pinned
digest. Its `live` job runs the pinned image under the manifest's posture and
takes every recording again. This repository holds no Python of its own.
`just ci` runs the gates that need nothing but this tree.

## Licence

The data in this repository is under the licence in `LICENSE`. ntfy is
dual-licensed by its authors under Apache-2.0 and GPL-2.0, and lemonfiber runs
it under Apache-2.0.
