# Contribution #1: Allow disabling sidebar tabs, topbar menu items, setting panels, etc. via hidden settings

Contribution Number: 1  
Student: Bolu Akande  
Issue: https://github.com/Comfy-Org/ComfyUI_frontend/issues/3388  
Status: Phase IV — PR Submitted

---

## Why I Chose This Issue

I chose this issue because it is part of the ComfyUI Frontend project, a widely used open-source tool in the AI ecosystem. The issue focuses on frontend architecture and configuration management, which aligns with my interests in software engineering, AI products, and building user-facing applications.

This issue also provides an opportunity to gain experience contributing to a large production codebase using modern technologies such as TypeScript and Vue. I hope to learn how large open-source projects structure frontend features, settings systems, and UI permissions while making a meaningful contribution to a tool used by thousands of users.

---

## Understanding the Issue

### Problem Description

Organizations that deploy ComfyUI Frontend as part of a hosted service may want to restrict access to certain functionality. Currently, there is no built-in way to disable specific sidebar tabs, topbar menu items, commands, or settings panels.

### Expected Behavior

Administrators should be able to configure hidden settings that disable selected UI functionality. Disabled features should not be accessible through the normal interface or through alternative workarounds.

### Current Behavior (before fix)

All supported functionality remains available to users, even in situations where organizations may prefer to restrict access to specific features. Disabled commands can still be triggered via keybindings and `commandStore.execute()`.

### Affected Components

- Sidebar navigation (`sidebarTabStore`)
- Topbar menu system (`menuItemStore`, hardcoded topbar components)
- Settings panels (`useSettingUI`, `settingStore`)
- Command registration and execution (`commandStore`)
- Frontend configuration management (`coreSettings`, `apiSchema`)

---

## Reproduction Process

### Environment Setup

| Tool | Version |
|------|---------|
| git | 2.49.0 |
| Node | v25.9.0 (from `.nvmrc` / `package.json` engines `>=25`) |
| pnpm | 11.3.0 (via corepack) |

```bash
cd ~/Desktop
git clone https://github.com/bluxio/ComfyUI_frontend.git
cd ComfyUI_frontend
git checkout fix-issue-3388
nvm install 25 && nvm use 25
corepack enable
pnpm install
pnpm dev:cloud   # http://localhost:5173
```

Fork: https://github.com/bluxio/ComfyUI_frontend  
Branch: https://github.com/bluxio/ComfyUI_frontend/tree/fix-issue-3388

### Steps to Reproduce (pre-fix behavior on `main`)

1. Start dev server with `pnpm dev:cloud` and open `http://localhost:5173/`.
2. Confirm sidebar tabs (Assets, Node Library, Workflows, etc.) are all visible.
3. Open **Comfy → File → Save** — save is available.
4. Press **Ctrl+S** — save still triggers via keybinding.
5. Open DevTools console and run `await window['app'].extensionManager.command.execute('Comfy.SaveWorkflow')` — command executes.
6. Run `await window['app'].api.getSetting('Comfy.UI.DisabledCommands')` on `main` — returns `undefined`.

### Reproduction Evidence

- Branch: https://github.com/bluxio/ComfyUI_frontend/tree/fix-issue-3388
- Implementation commits: `b2e73a698`, `7795187dd`
- Finding: No disable-list infrastructure existed before this PR; commands were the highest-leverage choke point.

---

## Solution Approach (UMPIRE)

### Understand

Organizations need hidden admin settings to disable UI features without end-user workarounds. Commands are the spine — menus, keybindings, and many buttons route through `commandStore.execute()`.

### Match

Reused existing patterns: `type: 'hidden'` settings (`Comfy.Extension.Disabled`), conditional UI gating (job-history sidebar tab), and menu built from command IDs.

### Plan

Phased delivery: (1) commands + menus, (2) sidebar tabs, (3) settings panels, (4) topbar catalog.

### Implement (this PR — Phase 1)

- Added `Comfy.UI.DisabledCommands` hidden `string[]` setting
- Added `isCommandDisabled()` utility
- Guarded `commandStore.execute()` for disabled commands
- Filtered disabled commands in `menuItemStore.registerCommands()`

### Review

Unit tests in `uiDisableList.test.ts`, `commandStore.test.ts`, `menuItemStore.test.ts`. All passing with `pnpm typecheck`.

### Evaluate

Delivers real value for "disable save" / "disable settings command" use cases. Sidebar, settings panel, and topbar disable lists remain follow-up work for #3388.

---

## Testing Strategy

```bash
pnpm test:unit src/platform/settings/utils/uiDisableList.test.ts \
  src/stores/commandStore.test.ts \
  src/stores/menuItemStore.test.ts
pnpm typecheck
```

Manual: set `Comfy.UI.DisabledCommands` to `['Comfy.SaveWorkflow']` via settings API, reload, confirm File → Save is hidden and Ctrl+S does nothing.

---

## Implementation Notes

Files changed:

- `src/platform/settings/constants/coreSettings.ts` — hidden setting definition
- `src/schemas/apiSchema.ts` — Zod schema
- `src/platform/settings/utils/uiDisableList.ts` — disable check utility
- `src/stores/commandStore.ts` — execute guard
- `src/stores/menuItemStore.ts` — menu filtering
- Tests for all of the above

---

## Pull Request

PR Link: https://github.com/Comfy-Org/ComfyUI_frontend/pull/13379

Status: Awaiting Review

Summary: Implemented the first phase of issue #3388 by adding hidden settings for disabling commands and preventing disabled commands from executing or appearing in menus.

---

## Learnings & Reflections

To be completed after maintainer review.

---

## Resources Used

- ComfyUI Frontend documentation
- GitHub issue #3388
- Project CONTRIBUTING.md
