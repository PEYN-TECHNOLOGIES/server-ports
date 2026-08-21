# server-ports

Registry of host ports in use on the PEYN VPS. Every project's deploy pipeline
checks `ports.csv` before publishing a new port, so two projects claiming the
same one fails the build instead of colliding silently on the server.

This repo is public deliberately — it holds no secrets, only port numbers and
project names, and that's what lets any repo's CI fetch it with a plain
`curl`, no cross-repo token needed.

## `ports.csv`

One row per **host** port actually published to the VPS (`docker run -p` /
compose `ports:`), covering all projects on the box — not just one.

| Column | Meaning |
|---|---|
| `Host Port` | The port as seen from outside the container |
| `Proto` | `tcp` or `udp` |
| `Binding` | `Public (0.0.0.0)` — reachable directly at `<server-ip>:<port>` — or `Loopback (127.0.0.1)` — only reachable from the host itself, typically fronted by nginx |
| `Project` | Which project owns this port |
| `Container` | The specific container name |
| `Container Port` | The port *inside* the container, before Docker's mapping |
| `Status` | `Up` or a note if it's currently broken |
| `Notes` | Anything worth knowing — plans to change the binding, shared resources, etc. |

## `internal-ports.csv`

Container ports that exist but are never published to the host — informational
only, can't cause a conflict, but useful context if one is ever about to be
published for the first time.

## Updating it

This registry is **checked, not auto-written** — a pipeline that wants a new
port fails loudly if that port is already claimed by a different project;
it never edits this file itself. Claiming a new port is a deliberate,
reviewed change:

1. Add a row to `ports.csv` in a PR (or direct commit, for a repo this small).
2. Merge it.
3. Only then deploy the project that uses it.

Doing it in the other order is exactly the race this registry exists to catch.

## How CI uses this

See `reusable-deploy.yml` in
[`PEYN-TECHNOLOGIES/peyn-media-service`](https://github.com/PEYN-TECHNOLOGIES/peyn-media-service)
— its `check-port` job fetches this file's raw URL and fails the deploy if
the `service_port` input is claimed by a different `project_name`. Any other
repo's pipeline can do the same: fetch
`https://raw.githubusercontent.com/PEYN-TECHNOLOGIES/server-ports/main/ports.csv`
and grep for the port you're about to publish.
