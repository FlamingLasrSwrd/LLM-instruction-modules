# Home Assistant: field notes for AI agents

Lessons from an AI agent managing a real Home Assistant instance over the
plain REST and WebSocket APIs, generalized so they apply to any instance.
Read beside the upstream best-practices skill in
[`best-practices/`](best-practices/SKILL.md), which covers design choices;
this file covers API mechanics and traps.

> **Last verified 2026-11-10** against Home Assistant core `dev` source
> (`components/api`, `components/history`, `components/config`), the REST
> API page and YAML style guide on developers.home-assistant.io (GitHub
> source), the `input_datetime` and `mcp_server` integration docs, and
> js-yaml 4.1.0 (the parser HA's frontend uses). Behaviour first observed on
> HA 2026.8 and 2026.9.

---

## 1. Connecting

- Authenticate with a **long-lived access token** (Profile → Security →
  Long-lived access tokens) sent as `Authorization: Bearer <token>`.
- Keep the URL and token outside the repository: environment variables or
  a scratch file with mode 600. Never commit, log, or echo the full token.
  If no valid token is available, ask the user for one rather than reusing
  a stale one.
- An instance on a private LAN address (`192.168.x.x`, `10.x.x.x`) is often
  unreachable from sandboxed browser tools, which block private ranges.
  Use `curl` from the shell, and if that is sandboxed too, check the
  sandbox's network rules before concluding HA is down.
- `POST /api/template` with `{"template": "..."}` is a read-only way to
  answer registry-style questions REST has no endpoint for:
  `area_entities('kitchen')`, `integration_entities('mqtt')`,
  `device_attr(...)`.

## 2. What REST can and cannot do

| Task | How |
|---|---|
| Read/write automations, scenes, scripts | REST: `GET`/`POST /api/config/{automation,scene,script}/config/<id>`, then the domain's `reload` action |
| Create or edit helpers (`input_boolean`, `input_number`, ...) | **No REST endpoint.** WebSocket `<domain>/create` and `<domain>/update`, or ask the user to create it in the UI |
| Read/write dashboards | **No REST endpoint.** WebSocket `lovelace/config` and `lovelace/config/save` (with `url_path` for non-default dashboards) |
| Integration health | REST: `GET /api/config/config_entries/entry?domain=<domain>` returns each entry's `state` (`loaded`, `setup_error`, `setup_retry`, ...) and `reason` |
| Error log | REST `GET /api/error_log` (admin), **but it is only registered when HA writes a log file.** Installs that log to the systemd journal (Home Assistant OS) return 404. Use Settings → System → Logs, the WebSocket `system_log/list` command, or ask the user to paste the log |
| History and logbook | REST `GET /api/history/period/<start>` and `/api/logbook/<start>`; see the trap below |
| Install an update (including HACS custom components) | `POST /api/services/update/install` with `{"entity_id": "update.<name>"}`, then usually a restart |
| Restart | `POST /api/config/core/check_config` first, then `POST /api/services/homeassistant/restart`. The API can still answer during shutdown, so confirm the restart by checking that something's `last_changed` has reset |

Ask the user before installing an update and before restarting.

## 3. Traps that cost a wrong diagnosis

- **History and logbook return only one day unless you pass `end_time`.**
  `end_time` defaults to `<start>` + 1 day, so a query starting five days
  ago silently drops the last four and looks like "nothing happened".
  Always pass `end_time=<now>`, URL-encoded (`+00:00` becomes `%2B00:00`),
  plus `minimal_response` to keep it small. Data older than the recorder's
  retention (`purge_keep_days`, 10 by default) does not exist.
- **"Unknown action" usually means the integration failed to load, not
  that the automation is wrong.** Check the config entry's `state` before
  editing YAML: `setup_error` means it crashed during setup and will not
  retry by itself (often fixed by updating the custom component);
  `setup_retry` means it is retrying, e.g. the device is unreachable.
  Cross-check `GET /api/services` for the domain.
- **An automation entity is not the toggle the user means.** A "mode" the
  user describes in plain words ("vacation mode", "working in the
  workshop") is usually an `input_boolean` or `input_select` that several
  automations react to, not an `automation.*` entity. Read the
  automations' triggers before deciding which entity belongs on a
  dashboard or gets toggled.
- **Restoring a previous state needs a snapshot helper.** Templates cannot
  look back in history. If an automation forces an entity into a state and
  must later restore what it was, save the old value into a helper
  (`input_boolean`, `input_text`) when forcing it, or use
  `scene.create` with `snapshot_entities`.
- **`input_datetime.set_datetime` with only `time:` leaves the date
  alone.** On a helper with both date and time, a daily `time:`-only call
  never advances the date. Pass `timestamp:` (a Unix epoch, e.g.
  `{{ as_timestamp(x) - 900 }}`) or `datetime:` to set both at once; or, if
  only a time of day is needed, create the helper with `has_date: false`.
- **Entity IDs that start with a digit** cannot use dot access in
  templates: write `states.switch['4ch_relay']`, or better,
  `states('switch.4ch_relay')`.

## 4. YAML the frontend will accept

The dashboard and automation editors parse YAML with js-yaml, so test what
you hand the user there, not only with PyYAML.

- Quote any scalar that **starts with** `&`, `*`, `!`, `%`, `@`, a backtick,
  `{`, `[`, `|`, `>`, `'` or `"`, and any that contains `: ` or ` #`.
  A leading `&` is the dangerous one: it parses without error as a YAML
  anchor and silently drops text (`title: &Humidity x` loads as
  `title: x`). A `&` mid-string (`Temperature & Humidity`) and Unicode
  box-drawing characters in comments are fine.
- Dashboard YAML has two different paste targets: the dashboard-level raw
  configuration editor expects the full config with a top-level `views:`
  list; a single view's own YAML editor expects only that view's fields
  (`title`, `path`, `cards`/`sections`, ...). Ask which one the user has
  open, or give the full `views:` form.
- Prefer creating automations, scripts and dashboards through the API over
  handing the user YAML to paste; see the best-practices skill.

### Condensed YAML style guide (developers.home-assistant.io)

- 2-space indentation; block-style lists indented under their key.
- Booleans only as lowercase `true`/`false` (never `yes`/`no`/`on`/`off`).
- Comments: capital letter after `# `, at the same indentation, above the
  line they describe.
- Avoid flow style (`[a, b]`, `{a: b}`); if used, `[1, 2, 3]` spacing.
- No explicit `null` or `~`; leave the value empty.
- Strings in double quotes. Exempt: entity IDs, attributes, device and area
  IDs, platform, condition and trigger types, action names, device
  classes, event names, and other fixed-choice values.
- Multi-line strings with `|` or `>` rather than `\n`.
- Omit options set to their default value.
- Use `target:` (`entity_id`/`device_id`/`area_id`) for actions, not a
  top-level or `data:` `entity_id`.
- `condition`, `action`, `sequence` and other mapping-or-list properties
  are always lists, even with one item. A property that takes a scalar or
  a list takes a single scalar for one value, not a one-item list.
- Templates: avoid them where plain YAML works; double-quote the outer
  string and use single quotes inside; spaces around `|` filters
  (`{{ x | float }}`); helper functions (`states()`, `state_attr()`,
  `is_state()`) instead of the `states.` object; split long templates over
  lines with `>-`; prefer the shorthand template condition
  (`conditions: "{{ ... }}"`).

## 5. MCP options

- [`homeassistant-ai/ha-mcp`](https://github.com/homeassistant-ai/ha-mcp):
  a community MCP server for configuring and automating HA from an AI
  assistant (same GitHub organization as the best-practices skill). A
  better fit than raw REST once you need WebSocket-only features such as
  helper or dashboard editing.
- HA's own [Model Context Protocol](https://www.home-assistant.io/integrations/mcp_server/)
  integration (since 2025.2) makes HA itself the MCP server. Its tools come
  in sets: **Device control** sees only the entities exposed to Assist,
  while **Home Assistant** can read all entities, logs and history and
  perform any action. The main URL exposes all sets and needs an
  administrator account; restrict them under the integration's Configure.
- The [Model Context Protocol Client](https://www.home-assistant.io/integrations/mcp/)
  integration works the other way: HA's conversation agent uses tools from
  another MCP server.

## 6. Reference links

- REST API: https://developers.home-assistant.io/docs/api/rest/
- WebSocket API: https://developers.home-assistant.io/docs/api/websocket/
- YAML style guide: https://developers.home-assistant.io/docs/documenting/yaml-style-guide/
- Automations: https://www.home-assistant.io/docs/automation/
- Troubleshooting automations: https://www.home-assistant.io/docs/automation/troubleshooting/
- Templating: https://www.home-assistant.io/docs/templating/
