# Publishing directly into Supernova (editor mode)

This reference documents the alternative to the local-markdown Save Option in Phase 4: writing the
generated documentation directly into Supernova's documentation tree, using the Supernova MCP's
admin write actions (`sn_admin_write` and friends). Use it once the user chooses "write into
Supernova" instead of (or in addition to) saving a local file.

This workflow only applies when the Supernova MCP is connected and exposes admin/editor tools
(`sn_admin_write`, `sn_admin_get_documentation_block_schema`, `sn_admin_get_entity_url`, etc.). If
those tools aren't available, fall back to the local-file Save Option.

**Before doing anything else, read the exact MCP resource `supernova://skills/use-supernova/SKILL.md`
and follow it.** It is the authoritative source for the write contract (one action per call, read
before every write, `patches` vs `mdx`, destructive-action confirmation rules, etc.). This file only
adds the parts specific to the supernova-doc skill: which template to duplicate, where to place the
new page, and the duplicate/rename/fill/verify/share sequence.

## 1. Choosing which template to duplicate

Supernova's documentation tree keeps at least one `_[Template] Component` tabbed-page group to
duplicate for every new component page. There are (at least) two variants:

- **`Components / _[Template] Component`** — use for a core Pixar component: anything that belongs
  directly under a category group inside `Components` (e.g. `Loading`, `Feedback`, `Actions`,
  `Layout`, `Modals`).
- **`Components / Satellite libraries / _[Template] Component`** — use for a component that belongs
  to a satellite library subgroup (e.g. `Marketing`, `Landings`, `Messaging`, `Monetization`). This
  template has fewer tabs (no separate `Content`/`Code` tabs seen so far) — confirm its current tab
  set with `sn_get_documentation_page_list` before relying on a fixed tab count, since template
  structure can change over time.

Ask the user: make sure the correct template will be used.

Find the current template group IDs and their child tab IDs with a paginated
`sn_get_documentation_page_list` call (follow every `nextCursor` — the list is long and template
groups are easy to miss on a single page). Do not assume tab titles or counts from a previous run;
re-read the template's actual tabs each time, since it can be edited independently of this skill.


## 2. Deciding where the new page belongs

1. Look at the component's Figma frame/category naming and the existing `Components` subgroups
   returned by `sn_get_documentation_page_list` (e.g. `Loading`, `Feedback`, `Actions`, `Layout`,
   `Modals`, `Satellite libraries/...`).
2. Place the new component next to its closest sibling by function, not just by name similarity.
   For example, a loading-indicator component belongs next to `Skeleton` under `Loading`, not under
   a generic catch-all group.
3. If no existing subgroup fits, or the right placement is genuinely ambiguous, ask the user rather
   than guessing or creating a new top-level group unprompted.
4. If the component belongs to a satellite library, nest it under the matching `Satellite libraries`
   subgroup (`Marketing`, `Landings`, `Messaging`, `Monetization`, ...), or ask if none fits.

## 3. Duplicate, rename, fill, verify, share

1. **Read first.** Call `sn_get_documentation_page_list` to get the template group ID, the target
   parent group ID, and (after duplicating) the new group's child tab IDs.
2. **Duplicate.** `DUPLICATE_DOC_GROUP` with `{ groupId: <template-group-id>, targetGroupId:
   <destination-group-id> }`. This duplicates the whole tabbed group, including every child tab
   page, in one call.
3. **Re-read to get new IDs.** The duplicate's pages are not returned by `DUPLICATE_DOC_GROUP`
   itself — call `sn_get_documentation_page_list` again (paginated) and match on the new
   `parentGroupId` to collect every duplicated tab's page ID.
4. **Rename.** Use `UPDATE_DOC_GROUP` to rename the duplicated group from the template's placeholder
   title (e.g. `_[Template] Component`) to the real component name, and `UPDATE_DOC_PAGE` on each
   duplicated tab to drop its placeholder prefix (e.g. `_Overview` → `Overview`). Leave the `_How to`
   tab's title and content untouched — it's meta-guidance for whoever fills the page, not
   per-component content, and is expected to stay hidden/prefixed.
5. **Inspect each template tab's body** with `sn_get_documentation_page_content` before writing,
   noting which catalog blocks it uses (e.g. `SNFigmaComponents`, `SNGuidelines`,
   `SNComponentChecklist`, `SNImage`). Call `sn_admin_get_documentation_block_schema` for each block
   package before writing it — don't guess attribute shapes.
6. **Write each tab's body** with `WRITE_DOC_PAGE_CONTENT` (`mdx` for a full rewrite, which is the
   normal case here since the whole template body is being replaced). Map the skill's four
   generated sections onto the template's tabs by content match, not by name: some templates split
   Overview/Usage/Content/Accessibility exactly as this skill does; others may organize differently
   (for example a Content tab that expects "Do/Dont" image-pair blocks per content category instead
   of the skill's tone/length/structure prose) — adapt the generated content into the target
   template's actual block structure rather than pasting the skill's markdown verbatim.
   - **Never change the template's block type.** If the template uses `SNFigmaComponents` /
     `SNFigmaComponent` (live component-frame previews) for a section, keep that block type in the
     filled page — do not substitute an `SNImage` picture embed just because a real Figma component
     ID is easier to reach than a resolved component ID. Swapping block types breaks the page's
     intended structure even if the substitute embed looks visually similar.
   - **`SNImage` is for genuinely standalone visuals only**, and only when the template itself calls
     for a picture rather than a component preview. Never point `figmaUrl` at a large or composite
     Figma frame (a "documentation showcase" frame, a frame with captions/callouts, or any per-variant
     frame that bundles several elements) — those are not going to render as clean embeds. Only use
     `figmaUrl` for a frame that is a single, isolated visual with nothing else in it. If no such
     frame exists, leave the `SNImage` slot empty (an `<SNImage />` with no `src`/`resourceId`/
     `figmaUrl`/`figmaSourceId`) rather than reaching for the nearest oversized frame.
   - `SNFigmaComponents` needs a resolved Figma-component ID from `sn_get_figma_component_list` /
     `sn_get_figma_component_detail`. Figma components are often imported under generic instance
     names (e.g. "Success", "Inverted") that are ambiguous across different design-system
     components — check whether the canonical component (`sn_get_component_detail`) has a linked
     "Figma component" property first, since that resolves one variant unambiguously; disambiguate
     any remaining candidates by comparing their `componentPropertyDefinitions` against the
     component's known variant axis (e.g. a Loader variant should expose the same `size: S/M`
     property as Loader's other confirmed variants — a candidate exposing an unrelated property
     like `platform` is very likely a different component that happens to share a generic Figma
     layer name). If you still cannot reliably disambiguate an item:
     - Inside a multi-item `SNFigmaComponents` grid, simply **omit that item** rather than adding an
       `<SNFigmaComponent id="" />` placeholder — Supernova rejects more than one empty `id` in the
       same block (`DuplicateEntityId`), so at most one slot in a grid can ever be truly empty.
     - For a section that has its own dedicated `SNFigmaComponents` block (one block per variant,
       as in a Usage page), leave that block self-closing with no child at all (e.g.
       `<SNFigmaComponents showProperties showName showDescription variant="grid" columns={1} />`)
       — this renders as an empty, unlinked component slot the design team can fill in later.
     - Never fall back to `SNImage` as a substitute in either case — tell the user which variant(s)
       are still missing a confirmed Figma component ID instead.
   - A tab outside this skill's scope (typically `Code`, which expects Storybook links and
     platform-specific snippets this skill never gathers) should not be filled with invented
     content. Either leave it as-is, or replace only clearly wrong/misleading placeholder content
     (e.g. a hardcoded link to a different component's Storybook story) with a neutral "not yet
     available" note, and tell the user this tab was intentionally left incomplete.
7. **Verify** by re-reading each written page with `sn_get_documentation_page_content` immediately
   after writing it, confirming no placeholder text remains and the body matches what was intended.
8. **Share the URL.** Get a shareable link with `sn_admin_get_entity_url`
   (`entityType: "docPage"`, plus the current `brandId` if the first call reports it missing) and
   share that URL with the user. Never share the design-system editor URL as the result.

## 4. Known caveats

- `sn_get_documentation_page_list` and `sn_search` can lag behind writes for **renamed** groups/pages
  (they reflect newly duplicated entities immediately, but a subsequent title rename may not show up
  in these read tools right away even though the write action reports success and the page's body
  content re-reads correctly). Don't treat a stale title in these tools alone as proof a rename
  failed — but do ask the user to confirm titles visually in the Supernova editor, since this
  skill's own read tools cannot fully verify it.
- Never call `PUBLISH_DOCS` or set an approval state beyond a plain draft unless the user explicitly
  asks for it. Treat `DELETE_*`, `PUBLISH_DOCS`, and approval-state changes as destructive actions
  requiring explicit confirmation, per the `use-supernova` skill's rules.
