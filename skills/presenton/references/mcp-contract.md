# Presenton Remote MCP Contract

The plugin connects to the Streamable HTTP MCP endpoint at:

```text
https://api.presenton.ai/skills/mcp
```

The MCP server advertises the authoritative JSON Schema for each tool. Clients must inspect those schemas rather than relying on undocumented fields. The following semantic contract keeps the skill and server aligned.

## Tools

### `search_designs`

Searches presentation designs using a natural-language visual query. It should accept a query and may accept a result limit. Each result should expose a stable design ID, title, and description sufficient to choose a design.

### `search_icons`

Searches public presentation-safe icons. It should accept a concept query and may accept a result limit and icon style/type. Results must provide reusable HTTPS image URLs.

### `import_public_image`

Imports an image that the user has authorized for external processing. The advertised schema may accept a public source URL or an encoded attachment with filename and media type. A successful result must provide a reusable HTTPS URL. Reject unsupported media types and oversized inputs with actionable errors.

### `validate_presentation_html`

Validates the complete HTML document against Presenton's exporter contract. It should accept the HTML and return a validity indicator plus actionable errors and warnings. Validation must cover the wrapper, direct 1280×720 slide children, forbidden embedded data URLs, supported Tailwind and Chart.js usage, asset URLs, and font imports.

The validator must reject every inline `style` attribute and every `<style>` element, including occurrences inside SVG or templates. It should also reject JavaScript that creates inline styles through `.style`, `cssText`, or `setAttribute("style", ...)`. The error should identify every offending line or a nearby source snippet when possible, so one correction pass can remove all occurrences. Callers must still perform the literal no-inline-style source audit described in `html-format.md` before invoking this tool.

The result must distinguish blocking `errors` from advisory `warnings`. `valid` is `true` whenever `errors` is empty, regardless of warnings. A missing or empty image `alt` value must be placed only in `warnings`—for example, `Image 1 should include non-empty alt text unless it is decorative.` It must not be added to `errors`, set `valid` to `false`, or block export. Empty `alt=""` is valid markup for a decorative image.

### `export_html_presentation`

Exports one validated HTML document to one requested format. It should accept the complete HTML, a single format (`pptx`, `pdf`, or `png`), and optional title and searched design ID. A successful result must contain a positive creation ID and an HTTP or HTTPS download URL.

### `create_presentation_preview`

Creates a shareable preview from a successful export creation ID. A successful result must contain an HTTP or HTTPS preview URL. Preview links are expected to expire after 24 hours.

## Shared response behavior

Any tool may return a non-empty `message` containing user-facing guidance. The skill relays that message verbatim. Errors should be actionable and distinguish invalid input, validation failure, unavailable service, and transient server failure.

## MCP tool annotations

Advertise accurate MCP annotations so Claude can group tools by risk:

- `search_designs`, `search_icons`, and `validate_presentation_html`: `readOnlyHint: true`, `destructiveHint: false`.
- `import_public_image`, `export_html_presentation`, and `create_presentation_preview`: `readOnlyHint: false`, `destructiveHint: false` because they create remote artifacts but do not overwrite or delete existing data.

Annotations describe behavior but do not bypass Claude's approval controls. Users decide whether tools run automatically through Claude's connector permission settings.
