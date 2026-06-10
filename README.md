# Triangle Agency Roll20 Sheet

This folder contains the first functional Roll20 conversion of the React/PDF sheet.
The conversion intentionally skips complex imagery and prioritizes Roll20-native
fields, repeating sections, and Sheet Workers.

## Roll20 Compatibility Rules

This folder is the source of truth for what is allowed in the generated Roll20
sheet. Changes to `triangle-agency.html` and `triangle-agency.css` must stay
inside Roll20's supported custom sheet model:

- Build the sheet as Roll20-compatible HTML and CSS, with JavaScript only inside
  `<script type="text/worker">` Sheet Worker blocks.
- Do not add React, Vite, browser app scripts, DOM event scripting, module
  imports, build-runtime dependencies, or service workers to the Roll20 sheet.
- Persisted character fields must use `name="attr_..."`.
- Simple roll buttons should use `type="roll"` with `name="roll_..."`.
  Rolls that need Sheet Worker math, such as QA/Burnout ability checks, should
  use `type="action"`, `name="act_..."`, `clicked:...` handlers, and
  Roll20 Custom Roll Parsing with `startRoll`/`finishRoll`.
- Repeating sections must use `fieldset` classes named `repeating_*`.
- Relationships are six fixed slots matching the source character sheet, not a
  repeating section.
- Requisitions/Benefits are six fixed slots matching the source character sheet,
  not a repeating section.
- Work/Life Balance tracks are fixed two-row travel paths: top-row attrs count
  left-to-right from 1 to 15, then bottom-row attrs continue right-to-left from
  16 to 30.
- Sheet logic must use Roll20 Sheet Worker APIs such as `on`, `getAttrs`,
  `setAttrs`, `getSectionIDs`, `generateRowID`, `removeRepeatingRow`,
  `startRoll`, and `finishRoll`.
- Do not rely on normal browser APIs inside Sheet Workers unless Roll20
  explicitly supports them.
- Tabs, toggles, and interactive controls must be implemented with normal
  Roll20-compatible form controls and CSS state, not browser JavaScript.
- CSS must be scoped to the sheet. Avoid styling the Roll20 app outside the
  sheet, and avoid generic class names that could collide with Roll20 defaults.
- Any new Roll20 API or HTML/CSS pattern must be either covered by the emulator
  or explicitly verified in Roll20's Custom Sheet Sandbox before we treat it as
  supported.
- CSS-drawn A/R/C triangle marks using `clip-path: polygon(...)` have been
  verified in Roll20 and are the preferred fallback for simple sheet art.
- Do not rely on CSS text warp effects, per-letter transforms, or text
  animation for sheet labels or roll templates; Roll20 removed this effect in
  sandbox testing. Use plain styled text instead.
- Do not use embedded `data:` image URIs for sheet art; Roll20 rejected them
  in sandbox testing. Use CSS-drawn marks or Roll20-compatible hosted assets.
- Character photos use Roll20's supported dynamic image binding:
  `<img name="attr_character_avatar">`. Do not replace this with browser
  upload/preview JavaScript or attribute interpolation in `src`.
- Do not reference local font files such as `C:\...`. Roll20 cannot load a
  user's local file path. Use a font-family stack with safe fallbacks, or verify
  an allowed hosted font URL in the Roll20 sandbox before relying on it.
- Roll20 supports Google Fonts imports for character sheets, but the documented
  supported host is `fonts.googleapis.com`. For CSE/default sheets, Google's
  generated `@import` rules can be used for sheet styling; roll templates may
  still need the legacy import form. For Legacy Sanitized sheets, load all
  Google fonts through a single `@import` and do not paste arbitrary generated
  URLs without sandbox verification.
- Avoid font names/imports containing `eval`; Roll20 documents this as a CSS
  filtering bug that can cause the stylesheet to be rejected.
- Never use `Candal` in the font stack. Headers should prefer the Avant Garde
  stack and fall back directly to common sans-serif fonts.
- Write-in fields use a typewriter-style monospace stack. This stays
  Roll20-compatible by relying on local/system fonts rather than local file
  paths or unverified hosted font URLs.
- Do not add external icon-font `<link>` rules to the generated Roll20 sheet.
  Google Material Symbols may be tested through a CSS `@import` from
  `fonts.googleapis.com`; each icon import must be verified in Roll20's
  sandbox before we rely on it broadly. The sheet currently tests Material
  Symbols for `edit_note`, `emoji_events`, `gavel`,
  `local_fire_department`, `lock`, and `workspace_premium`; keep labels
  visible so the sheet remains usable if the external font fails to load.
- QA MAX controls use `type="text"` plus numeric entry hints instead of
  `type="number"` so browsers do not render spinner arrows over the triangle.
- QA current controls are nine same-name Roll20 radio inputs styled as CSS
  triangles. The stored attrs remain `*_current`, so Sheet Worker burnout
  calculations consume the same values as before.
- Ability roll actions use Custom Roll Parsing to count natural 3s and apply
  current Burnout. QA spending is intentionally manual for now.
- Ability roll output uses Roll20 template conditionals and CSS-drawn triangle
  d4 faces. Natural 3s render as filled red triangles; other d4 faces render as
  outlined triangles. Burnout-marked 3s render with an X overlay.
- For Custom Roll Parsing, values supplied through `finishRoll` must be read
  in roll templates as `computed::<rollname>`. Plain roll names such as
  `die_1` only refer to the placeholder inline rolls sent to `startRoll`;
  Roll20 will not use the `finishRoll` replacement value unless the template
  references `computed::die_1`, `computed::success_flag`, etc.
- Rolltemplate chat CSS is fragile in Roll20's sandbox. The emulator can apply
  `.sheet-rolltemplate-...` CSS successfully while Roll20 chat still renders
  the template as plain text or only partially styled. Critical roll output
  styling should therefore have inline `style` fallbacks inside the
  `<rolltemplate>` markup. Keep external rolltemplate CSS as an enhancement,
  not the only rendering path.
- Rolltemplate result icons should avoid relying on inline `clip-path` or
  nested checkmark CSS. Roll20 chat rendered result icons as incorrect square
  or partially-filled shapes when they used `clip-path`, and the success
  checkmark did not survive reliably. Use older border-built shapes for
  critical result symbols; the success state uses a plain centered `SUCCESS`
  label below the triangle.
- Ability Question fields are structured as Be Known tracks: Sheet Workers
  populate readonly prompt/answer/code attrs, and the three boxes beside each
  answer are ordinary persisted `attr_...` checkboxes. The current local
  anomaly rules contain the prompt, answers, and unlock codes; keep those values
  in `src/rules/anomalies.js`, not in the Roll20 generator.

### Roll20 Good Code Requirements

- Maintain enough CSS/HTML styling for the sheet to be aesthetically usable.
  Layouts must be understandable to players familiar with the paper sheet, must
  prioritize ease of use, and must not unintentionally overlap when the Roll20
  window is resized.
- Use proper HTML syntax and maintainable container elements such as `<div>`
  and `<span>`. Do not include `<head>` or `<body>` tags in the Roll20
  sheet HTML because they can prevent loading in the virtual tabletop.
- Do not use `<table>` for layout. Tables are only acceptable for actual
  tabular data.
- Keep submitted HTML, CSS, and JSON files Unix-compatible with LF line endings.
  Public Roll20 submissions must include a valid `sheet.json` and preview
  image.
- Test functionality and styling in Roll20's supported browsers: Chrome and
  Firefox.

### Roll20 Satisfactory Experience Requirements

- The sheet must be standalone by default. Basic functionality must work without
  external images, external fonts, or Mod/API companion scripts. Companion
  scripts may supplement the sheet, but must not be required.
- Respect existing Roll20 sheets for the same game. Before publishing a new
  community sheet, check whether one already exists. If replacing an unmaintained
  sheet, preserve/convert existing attributes where possible so existing
  campaign data is not erased.
- Include functional roll buttons for common game rolls where the system uses
  rolls. If a roll concept does not apply, document that in the submission notes.
- Use persisted `<input>` elements for character attributes/stats and
  `<textarea>` elements where users need notes or descriptions. Prefer clear,
  spelled-out attribute names and, when relevant, names compatible with existing
  sheets for the same system.
- Ensure the game rules are readily available online to the public, and do not
  embed copyrighted rules text beyond what is allowed for the sheet.

If a requested feature cannot be implemented within these constraints, we should
push back before coding it. Acceptable pushback should name the unsupported
piece, explain the Roll20 constraint, and offer the closest Roll20-compatible
alternative.

References:

- [Roll20 Intro to Sheet Development](https://help.roll20.net/hc/en-us/articles/360037773413-Intro-to-Sheet-Development)
- [Roll20 Sheet Worker Scripts](https://help.roll20.net/hc/en-us/articles/360037773513-Sheet-Worker-Scripts)
- [Roll20 Repeating Sections](https://wiki.roll20.net/Character_Sheet_Development/Repeating_Section)
- [Roll20 Character Sheet HTML](https://wiki.roll20.net/Character_Sheet_Development/HTML)
- [Roll20 CSS Wizardry: Google Fonts](https://wiki.roll20.net/CSS_Wizardry#Google_Fonts)

Files:

- `triangle-agency.html`: Roll20 sheet HTML plus Sheet Worker data binding.
- `triangle-agency.css`: Roll20 custom sheet CSS.
- `preview.html`: local browser preview wrapper; Sheet Workers only run in Roll20.
- `sheet.json`: Roll20 sheet metadata stub.

Regenerate from the repository root:

```powershell
npm run build:roll20
```

The generator reads rule data from `src/rules`, so Anomaly, Reality, and
Competency changes should be made there first.

Recommended Roll20 testing flow:

1. Use the Roll20 Sheet Sandbox for testing.
2. Upload or paste `triangle-agency.html` and `triangle-agency.css`.
3. Confirm selector changes populate the derived rules fields.
4. Confirm the six fixed Relationship slots save.
5. Confirm the six fixed Requisition/Benefit slots save.
