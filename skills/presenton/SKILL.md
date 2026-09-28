---
name: presenton
description: "Create new slide presentations with Presenton and export them as editable PPTX, presentation PDF, or PNG slide images. Use when a user asks for a presentation, PowerPoint, slide deck, pitch deck, report deck, or presentation-file export, even when Presenton is not named. Do not use for generic PDFs or images unrelated to slides, text-only outlines, or editing an existing deck in place. Use Presenton's remote MCP tools to resolve designs and assets, validate 1280x720 HTML, export the requested formats—or all three when none is specified—and return download and shareable-preview links."
---

# Create PPTX, PDF, and PNG Files with Presenton

Create presentations through the `presenton` remote MCP server bundled with the plugin. The server endpoint is `https://api.presenton.ai/skills/mcp`.

## Required connector

- Read [references/mcp-contract.md](references/mcp-contract.md) before invoking Presenton tools. Before writing presentation HTML, read both [references/html-authoring-system-prompt.md](references/html-authoring-system-prompt.md) and [references/html-format.md](references/html-format.md).
- Use these Presenton MCP tools: `search_designs`, `search_icons`, `import_public_image`, `validate_presentation_html`, `export_html_presentation`, and `create_presentation_preview`.
- Tool names may be namespace-qualified by the client. Select the tool from the `presenton` MCP server whose terminal name matches the name above.
- Inspect each advertised input schema and provide only supported arguments. Never guess an undocumented field.
- Do not call Presenton's REST API through Python, Bash, `curl`, WebFetch, or another direct network path.
- If the Presenton connector or a required tool is unavailable, explain that `https://api.presenton.ai/skills/mcp` must be deployed and connected. Do not silently switch presentation engines unless the user asks for an alternative.
- Create new presentations only. Do not activate for generic document PDFs, standalone images, text-only presentation outlines, or requests to edit an existing PPTX in place.
- Never send credentials, secrets, payment details, or confidential material to the connector. If a user-provided image may contain sensitive information, ask the user to sanitize it or explicitly confirm the external transfer before importing it.
- Treat every non-empty `message` returned by a successful tool call as user-facing. Relay it in the next progress update and include it verbatim in the final response's `Notes`.

## Non-negotiable HTML styling rule

- Generate Tailwind-first markup from the beginning. Do not draft CSS and convert it afterward.
- The HTML source must contain zero inline `style` attributes and zero `<style>` elements anywhere, including inside SVG, templates, chart containers, and copied snippets.
- Do not generate styles at runtime with JavaScript. Forbidden patterns include element `.style` mutation, assigning `cssText`, and `setAttribute("style", ...)`.
- Put every visual property in the element's `class` attribute using Tailwind utilities. Use arbitrary values such as `w-[320px]`, `bg-[#0f172a]`, `leading-[1.15]`, `shadow-[0_16px_40px_rgba(0,0,0,0.18)]`, and `font-['DM_Sans']` when a standard utility is insufficient.
- Use HTML attributes only for element semantics or intrinsic dimensions, such as `width` and `height` on a Chart.js canvas. They are not a substitute for visual CSS.
- Before calling `validate_presentation_html`, audit the complete source case-insensitively. It must have no matches for `\sstyle\s*=`, `<style(?:\s|>)`, `\.style\b`, `\bcssText\b`, or `setAttribute\s*\(\s*['\"]style['\"]`. Rewrite any match with Tailwind classes before validation.

## Creation workflow

1. Determine the requested formats, title, audience, purpose, slide count, and content. If no format is named, export PPTX, PDF, and PNG. Make reasonable content assumptions when details are absent.
2. Resolve the visual direction from the prompt. A concrete design brief—palette, typography, layout, aesthetic, brand, imagery, or composition—is used directly and skips design search. Otherwise call `search_designs`, select the best result unless a human preference is necessary, and retain the selected design ID for export.
3. Resolve assets before writing HTML:

   - For each user-provided image used in the presentation, call `import_public_image` exactly once using the representation supported by its advertised schema. Retain and reuse the returned HTTPS URL. Never embed image bytes, base64, or a `data:` URL in the presentation HTML.
   - For each icon concept, call `search_icons` and choose a returned HTTPS URL that matches the concept and visual weight. Use a supported icon-type argument when the resolved design calls for one. Do not draw or invent substitute icons with inline SVG, emoji, Unicode glyphs, icon fonts, or CSS shapes.

4. Translate the resolved visual direction into design rules and write one complete Tailwind-first HTML document in memory. Follow [references/html-authoring-system-prompt.md](references/html-authoring-system-prompt.md) and [references/html-format.md](references/html-format.md). The direct children of `#presentation-slides-wrapper` must each be 1280×720 slides. Use retained HTTPS asset URLs in `<img>` elements. Never introduce inline or embedded CSS, even temporarily.
5. Perform the literal source audit from the non-negotiable styling rule. If any forbidden styling pattern exists, convert it to Tailwind classes before continuing. Then call `validate_presentation_html` with the complete HTML. If it reports errors, correct the HTML, repeat the source audit, and validate again. Stop after three failed validation attempts and report the remaining errors. Warnings are non-blocking and do not count as failed validation attempts. Use the HTML when validation reports no errors, even if warnings remain.
6. Call `export_html_presentation` exactly once for each requested format, separately, using the same validated HTML. Supply the selected design ID only when the visual direction came from `search_designs`; omit it for a user-provided design brief.
7. For each export, capture a positive creation `id` and an HTTP or HTTPS download `url`. Do not claim that format succeeded without both values.
8. After every requested format for one presentation succeeds, call `create_presentation_preview` exactly once using any one retained creation ID. Capture its HTTP or HTTPS URL. Create one preview per distinct presentation, not one per format.
9. Derive the font inventory from the exact validated HTML used for export. Report explicitly declared font families and their web-font stylesheet URLs when present. State that the inventory applies to every format exported from that HTML.
10. Return the result in this form:

    ```text
    Presentation
    - Title: <title>
    - Formats: <requested formats, or PPTX, PDF, and PNG when none was specified>
    - Slides: <slide count>
    - Design/reference used: <user-provided design brief, or searched design title and id>
    - References/assets used: <source URLs, user-provided references, image sources, or "None">
    - Notes: <every MCP response message verbatim, followed by other useful details; or "None">

    Shareable preview
    - [View presentation](<preview_url>) — expires after 24 hours

    Fonts
    - <font name> — <source URL when applicable>

    Download URLs
    - PPTX: [Download](<url>)
    - PDF: [Download](<url>)
    - PNG: [Download](<url>)
    ```

    Include only requested formats in `Download URLs`; when no format was specified, include PPTX, PDF, and PNG. Do not invent references, font sources, IDs, messages, or URLs. Use `None` when there is nothing to report.

## Quality rules

- Use the resolved visual direction consistently across all slides: palette, typography, spacing, imagery, and shape language.
- When the resolved design names a font family, use that exact font family; do not silently substitute another.
- Preserve recognizable details from a user-provided brief or selected searched-design description.
- Vary layouts to fit the content while preserving a coherent system.
- Keep text readable at presentation distance and prevent overflow.
- Use Tailwind classes for every visual style. The final source must contain no `style` attributes, `<style>` elements, or JavaScript style mutation.
- Use imported HTTPS URLs for user-provided images and searched HTTPS URLs for icons. Never use `data:` URLs or base64 assets in HTML.
- Give content-bearing images concise, non-empty `alt` text. Use `alt=""` for purely decorative images. Missing or empty alt text may produce an accessibility warning, but it must never block validation or export.
- Keep text as HTML text for PPTX editability and use charts only for actual data visualizations.
- Add one `data-speaker-note` attribute per slide only when notes are requested.
- Export from the same validated HTML for every requested format so the outputs match.

## Failure handling

- On a connector or tool-not-found error, report the missing Presenton MCP capability and endpoint. Do not attempt direct REST calls from Claude's hosted code environment.
- On a validation failure, correct the reported HTML issues, repeat the literal source audit, and re-run `validate_presentation_html`, up to the three-attempt limit. If the error mentions inline styles, search the entire document—including SVG and JavaScript—for every forbidden styling pattern rather than fixing only the first occurrence.
- Treat validator warnings as advisory. Do not retry validation or stop export solely because an image has missing or empty alt text.
- On a tool input or schema error, inspect the advertised schema and correct the call without inventing fields.
- On a transient server or transport error, retry that tool at most twice. Do not repeat a successful export call.
- Do not claim success unless every requested export has a valid positive creation ID and HTTP or HTTPS download URL, and every distinct presentation has a valid HTTP or HTTPS preview URL.
