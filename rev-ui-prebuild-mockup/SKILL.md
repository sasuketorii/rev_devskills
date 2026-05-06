---
name: rev-ui-prebuild-mockup
description: Use when planning RevSystem or RevLicense UX/UI implementation by creating DESIGN.md-constrained pre-build mockups, screenshots, and implementation briefs before coding dashboards, admin consoles, portal pages, sidebar flows, or dense operational UI.
---

# Rev UI Prebuild Mockup

Use this skill before coding UX/UI when the user wants to turn a workflow, screen, dashboard, admin console, portal page, or role-based state into a buildable mockup package.

The generated mockup is only a layout-intent capture. `DESIGN.md` and existing code remain the source of truth.

## Workflow

1. Read the project design source of truth first:
   - `DESIGN.md`
   - `AGENTS.md`
   - Any route-specific docs or existing UI notes the repo already uses
2. Inspect the existing application shell, sidebar, route structure, components, tokens, and page patterns before proposing replacements.
3. For RevLicense, reuse the existing `PortalShell` and sidebar unless the user explicitly asks for shell/sidebar redesign.
4. Use references, screenshots, and third-party products only for IA, interaction density, hierarchy, and layout ideas. Never copy brand, colors, typography, icon style, names, data, or visual identity.
5. Write page IA and an implementation brief before generating any image:
   - target route, role, state, viewport, and theme
   - purpose and primary task
   - page regions and hierarchy
   - component mapping to existing components or planned components
   - data contract: required fields, permissions, mutations, loading/error/empty states
   - locale strategy for ja/en/zh/ko, including copy keys or copy tables instead of hardcoded implementation copy
   - longest-label and CJK wrapping risks for buttons, tabs, tables, sidebars, badges, empty states, dialogs, and other text-bearing UI
   - security notes, including destructive-action handling
   - acceptance criteria and screenshot/test verification plan
6. Generate pre-build mockup images as layout-intent captures, not design authority. A mockup image may use one locale, but the implementation brief must assume native UX readiness for Japanese, English, Chinese, and Korean from the start. The prompt must cite `DESIGN.md` constraints and existing shell/sidebar reuse.
7. Review each generated image before saving:
   - `DESIGN.md` drift: colors, density, typography, radii, spacing, light/dark behavior
   - shell/sidebar drift, especially RevLicense `PortalShell`
   - multilingual readiness: text expansion/contraction, CJK line wrapping, native-length labels, and no viewport-scaled font hacks
   - implementation usefulness: can a coder map it to components and states?
   - PII/secrets/project-identifying data exposure
8. Sanitize or redact images before saving approved captures. Discard images that cannot be made safe or useful.
9. Save only approved artifacts under the target project:
   - `docs/ui-mockups/YYYY-MM-DD/`
   - filename format: `<route-slug>--<role>--<state>--<locale>--<theme>--<viewport>.<png|md>`; if an external capture tool cannot use this format, record locale, theme, and viewport in adjacent metadata
   - include the markdown brief beside the mockup image
10. Require reviewer LGTM before handing the brief and captures to a coder.
11. During implementation, treat the brief as the build contract and the PNG as a visual aid only.
12. Final implementation must be verified with live screenshots and relevant tests before completion.

## Safety Rules

- Never include secrets, real emails, real domains, Supabase project refs, full IDs, IP addresses, hostnames, tokens, license keys, customer names, or production data in generated or saved images.
- Use fake placeholders such as `user@example.test`, `tenant_demo`, `lic_****1234`, and `example.test`; redact before saving if needed.
- `DESIGN.md` is authoritative. A generated PNG must never override tokens, brand, accessibility requirements, shell rules, or component behavior.
- Preserve the existing shell/sidebar unless the task explicitly includes a shell redesign.
- Destructive actions require an AlertDialog-like confirmation in the UI plan and a server-side authorization note in the implementation brief.
- Treat Japanese, English, Chinese, and Korean as native product locales, not literal afterthought translations. Labels should be product-native per locale and must avoid culturally awkward literal copy.
- Do not use layout tricks that hide multilingual problems, such as viewport-scaled font hacks or truncation that prevents task completion.

## Output Checklist

Before coding handoff, provide:

- approved mockup image path
- markdown brief path
- route, role, state, theme, and viewport covered
- locale covered by each image and ja/en/zh/ko strategy covered by the brief
- copy keys or copy table, plus longest-label risk notes
- component mapping
- data contract and state coverage
- security notes
- acceptance criteria
- explicit reviewer LGTM status
