# Magewire Agent Skills

Portable skills for developing, extending, and maintaining [Magewire 3](https://github.com/magewirephp/magewire).

Each top-level directory is a standalone skill. Keep its complete directory together so any bundled `references/` remain available to the agent.

| Skill | Use it for |
|---|---|
| `magewire` | Components, properties, actions, lifecycle, events, and directives |
| `magewire-architecture` | Framework internals, Features, Mechanisms, request handling, and extension points |
| `magewire-backwards-compatibility` | Migrating Magewire 1 components and controlling the V3 compatibility layer |
| `magewire-best-practices` | Implementation and review guidance for production Magewire code |
| `magewire-javascript` | Magewire, Alpine, CSP, and JavaScript integration |
| `magewire-portman` | Syncing Magewire's ported Livewire source |
| `magewire-theming` | Theme compatibility modules, Hyvä, Hyvä Checkout, Tailwind, and admin boundaries |

## Compatibility

The repository follows the open [Agent Skills specification](https://agentskills.io/specification): every skill is a directory with a `SKILL.md` entrypoint and optional relative resources. The canonical skills contain no client-specific manifests, tool names, permission grants, or installation paths.

Use the discovery or installation mechanism documented by your agent client and install each selected directory intact. Agents without native Agent Skills support can still use the Markdown instructions as context, provided referenced files remain accessible.

Execution permissions remain under the control of the agent client and the user; these skills do not pre-authorize commands or repository changes.

The skills complement the [Magewire documentation](https://docs.magewirephp.nl/). The tagged source and release history in `magewirephp/magewire` remain authoritative when a skill, documentation page, or third-party integration disagrees with the framework.

## Contributing

Keep guidance specific to Magewire. Link to the [Livewire 3 documentation](https://livewire.laravel.com/docs/3.x/) for behavior inherited unchanged from Livewire instead of duplicating its full reference material.
