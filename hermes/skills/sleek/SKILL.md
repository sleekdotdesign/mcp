---
name: sleek
description: Design mobile app screens in Sleek through the sleek MCP server, show them to the user, and hand back the project link to open, edit, and export.
version: 1.0.0
---

# Designing with Sleek

Sleek (https://sleek.design) designs mobile app UI with AI. The `sleek` MCP server turns a plain-language description of an app into real, editable screens. Use it whenever the user asks to design, mock up, or redesign mobile app screens, or to build an app from Sleek designs.

## Workflow

1. **Pick a project.** Call `list_projects` to reuse one the user names, or `create_project` for a new app.
2. **Send the whole app as ONE `design` call.** Put every screen in a single brief: "onboarding, home, item detail, cart, and profile for a plant shop app, warm minimal style". Sleek's design agent plans the screen set together, so one call gives a coherent app. Separate calls give unrelated screens and spend credits each time.
3. **Show the result.** When the run completes, call `show_project` with its `runId` to show the user what the run made. Use `screenshot` only where the client cannot show apps; it renders up to four screens per call, so render more in batches.
4. **Always share the `projectUrl`.** That link is where the user opens, edits, and exports the designs to Figma, HTML, or React. Send it every time; do not just describe what was made.
5. **Build from the design.** To build the working app, call `get_component` for each screen and use its HTML as the design source.

## Editing a screen

Pass the screen's `screenId` to `design` with the change, for example "move Add to cart into a sticky bottom bar". Every screen has two ids: `screenId` (the frame on the canvas, used by `design`) and the component `id` (used by `get_component` and `screenshot`). Never pass a component id as `screenId`.

## Credits

`design` spends the workspace's AI credits: about 30 per new screen, 5 to 30 per edit. Listing, reading, showing, and rendering are free, so browse freely. Hermes asks the user before each `design` call.

- Pass an `idempotencyKey` so a retry returns the original run instead of charging again.
- If a run or call fails with a plan or credit limit, relay the explanation as written and send the user to the URL it includes (usually https://sleek.design/pricing).

## Runs

- One design run at a time per project. On a conflict, poll `get_design_run` until the current run settles (completed or failed), then retry.
- `design` waits up to about 270 seconds. If it returns `running`, poll `get_design_run`.
- `cancel_design_run` asks a run to stop sooner. Credits already spent are not refunded.

## Deleting

`delete_project` permanently deletes a project and every screen in it. Only call it after the user has named the exact project to delete in this conversation, and pass `confirm: true`. Never delete to clean up, retry, or make room on your own initiative.

## Sign-in

Hermes handles sign-in. If a tool returns 401 or says authentication is required, tell the user to run `hermes mcp login sleek` and sign in to Sleek in the browser. If a tool result includes a URL for the user to act on, send it to them.
