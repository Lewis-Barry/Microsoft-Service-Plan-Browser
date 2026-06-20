[TEXT]
- No comparatives framed as "X is not Y, it is Z instead", state what something is directly
- Positive, factual tone throughout
- No Em-dashes in any generated material

[Strict Design Rules]
- No emoji in design used in text, placeholders, or enlarged as icons
- No rounded corner card layouts in a grid which have object layout pattern: icon, heading, small text desc.
- Straight lines and hard corners preferred in UI
- Do not use fonts: Segoe UI, Outfit, Droid Sans, Inter, Barlow
- No excessive on-hover animations for UI elements
- UX should be subtle, require no explanation
- Do not use small block capitals to segment or use as headings for design elements
- Iconography MUST be fetched from https://fontawesome.com/ (free icons) or https://www.svgrepo.com/. Do not hand-draw or invent SVG icons. Embed the fetched SVG markup inline (no CDN link), recolour to the palette, and state in your summary which icons you pulled and from where. If a source is unreachable, stop and ask rather than substituting hand-drawn shapes.
  - Note: fontawesome.com icon pages do not expose raw SVG markup to automated fetches. Pull Font Awesome Free icons directly from the source repo instead: `https://raw.githubusercontent.com/FortAwesome/Font-Awesome/6.x/svgs/solid/<icon-name>.svg` (also `regular/` and `brands/` folders). These are the canonical FA Free icons. For svgrepo, use the per-icon download URL `https://www.svgrepo.com/download/<id>/<name>.svg`.
- Data visualisations must be immediately self-evident without explanation. If a visual element requires context or inference to interpret, replace it with a labelled value.
- Do not use small, text "pills" to indicate status of anything whether rectangular or rounded - be smarter with colours instead or use text if there aren't enough in the palette
- Follow Gestalt Principles for size and positioning of elements/features


[Accessibility]

Foreground/background/text/elements MUST follow WCAG 2 guidelines:

Perceivable
Operable
Understandable
Robust

**And contrast rules:**
The visual presentation of text and images of text has a contrast ratio of at least 4.5:1, except for the following:

***Large Text***
Large-scale text and images of large-scale text have a contrast ratio of at least 3:1;

***Incidental***
Text or images of text that are part of an inactive user interface component, that are pure decoration, that are not visible to anyone, or that are part of a picture that contains significant other visual content, have no contrast requirement.

***Logotypes***
Text that is part of a logo or brand name has no contrast requirement.

[Colours]

Use this colour pallette:

$tomato: #f06543ff;
$platinum: #e8e9ebff;
$soft-linen: #e0dfd5ff;
$gunmetal: #313638ff;
$sandy-brown: #f09d51ff;

([paste from cooolers](https://coolors.co/))

[Colour Roles]
Before using the palette, assign each colour a single role and reuse it
consistently. Roles, not specific colours:
- Primary accent: structure, links, default icons, active states
- Secondary accent: must be from the same temperature/family as the primary;
  used for supporting emphasis
- Neutral(s): surfaces, borders, dividers
- Text: the highest-contrast colour against the background
- Semantic-only: any remaining colours that contrast with the accents are
  reserved for meaning (warning, alert, error) and must not be used as neutral
  category labels

Rules:
- Every functional icon defaults to the primary accent unless it carries its
  own semantic meaning.
- A set of related items (statuses, categories, tabs) must draw from ONE colour
  family. Do not mix a colour from outside the accent family into the same
  control group.
- Decorative/icon colour must visually relate to the nearest structural colour
  (same family), never be a standalone outlier.
- If a category set has more members than the family has distinct colours,
  differentiate the extras with text or iconography, not by reaching for an
  unrelated hue.
- If the palette lacks an obvious semantic colour, express status with text and
  iconography rather than borrowing an accent colour.
