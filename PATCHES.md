# Viper's html5ever fork: patch ledger

This repository is a fork of [html5ever](https://github.com/servo/html5ever)
kept by the Viper project. Its `viper/*` branches carry a small set of
patches on top of exact html5ever release commits; `main` and every other
branch are not Viper's and follow their own history. Viper builds
`html5ever` from one exact commit of a `viper/*` branch, through a Cargo
`[patch.crates-io]` entry; every other package of this workspace comes to
Viper from crates.io (the patch `registry-deps`).

This file is the canonical ledger of those patches. It holds the verified
base, one row and one section per patch, the fork-side rebase procedure and
the rebase log. Viper's own record of the fork (how the pin moves, the gate
step that checks it, the cargo-deny source and the licence obligations) is
its ADR 0035, "The html5ever fork".

Base: d7232d746d3112f08926d169591576bd12fe2fdf
Branch: viper/html5ever-0.40.1

## Rules

- **Branches.** One long-lived branch per html5ever release,
  `viper/html5ever-<version>`, created at the commit the published crate
  names. A `viper/*` branch is never force-pushed, and never deleted while
  any Viper revision pins a commit on it (a repository ruleset blocks both).
- **Markers.** Every changed hunk carries `VIPER-PATCH(<id>)` in a comment
  (`//` in Rust, `#` in TOML), with `<id>` matching `[a-z0-9-]+` and naming
  a row of the ledger below. The comment states the specification followed
  or the problem fixed, and why upstream differs, with a specification link
  where one exists. Every file that differs from the base, apart from this
  one, carries a marker. Viper's gate step `html5ever-fork` fails when the
  marker ids in the pinned commit differ from the ledger ids, or when a
  changed file carries no marker.
- **Size.** Each patch is small (target at most 100 changed code lines) and
  self-contained, one commit per patch where possible.
- **Tests.** Every patch has tests in the fork (listed in its row) and a
  proof in Viper's gate: named Viper tests that must pass in the gate's run.
- **Upstream.** A patch may be offered upstream later; nothing goes to any
  servo repository (issue, pull request or comment) without the Viper
  owner's go-ahead for that specific draft. A patch is dropped when
  upstream releases an equivalent.
- **Licences.** html5ever is MIT OR Apache-2.0 (`LICENSE-MIT`,
  `LICENSE-APACHE`, copyright the html5ever Project Developers); patched
  files keep their notices, the licence files are unchanged, and each row
  names the licence of its change.
- **Commits** use the Viper owner's GitHub noreply identity.

## Base

`viper/html5ever-0.40.1` starts at html5ever commit
`d7232d746d3112f08926d169591576bd12fe2fdf` ("Bump version number to 0.40.1
(#786)"). The published crates Viper builds were verified against this
repository on 2026-10-24:

| Crate | Published `.crate` SHA-256 (equals Viper's `Cargo.lock` checksum) | `.cargo_vcs_info.json` | Sources compared with the base |
| --- | --- | --- | --- |
| `html5ever` 0.40.1 | `456a1a377e608e555d22ddab27ac0114bc7a7b4199078108e34c2aeae6c9b130` | `sha1` `d7232d746d3112f08926d169591576bd12fe2fdf`, `path_in_vcs` `html5ever` | identical: every packaged file under `html5ever/`; `Cargo.toml.orig` = `html5ever/Cargo.toml`; the packaged `README.md` = the root `README.md` (its `readme = "../README.md"`). Not packaged: `html5ever/fuzz` |
| `markup5ever` 0.40.0 | `0ab3dc68ac4a0f5719e560136778c1ee716e296030d75dbd4484e37e39e3a842` | `sha1` `a193ea7f2492d1e51eb32955a09c241345348dba`, `path_in_vcs` `markup5ever` | identical to the base's `markup5ever/` (and to `a193ea7f`'s): every packaged file and `Cargo.toml.orig` |
| `web_atoms` 0.3.0 | `7572660c8890448ba236b7376f27e389c6a7e1c70195622faced601f855c0ada` | `sha1` `a193ea7f2492d1e51eb32955a09c241345348dba`, `path_in_vcs` `web_atoms` | identical to the base's `web_atoms/`: every packaged file and `Cargo.toml.orig` |
| `tendril` 0.5.1 | `5fed54709c5b3a53d09bb1c113ea4f5ceafd1e772ddcb0030a82e1d56c087b08` | `sha1` `d64dfd4c21cf2451649107ade7eaf042d95fbc5a`, `path_in_vcs` `tendril` | **differs** from the base's `tendril/`: `src/buf32.rs`, `src/stream.rs`, `src/tendril.rs`, `examples/fuzz.rs` and `Cargo.toml` changed after 0.5.1 was published |

Generated at packaging time and not compared: the normalized `Cargo.toml`,
`Cargo.lock` and `.cargo_vcs_info.json`. Because `tendril` differs, a git
dependency on this workspace as published would build a `tendril` (and,
through the workspace's path dependencies, a `markup5ever` and `web_atoms`)
that Viper does not pin. The patch `registry-deps` makes the workspace take
those three from crates.io, so that Viper takes only `html5ever` from here
and `Cargo.lock` changes only html5ever's source line.

## Ledger

| Id | Reason | Register entry | Tests | Licence and upstream attribution | Upstream status |
| --- | --- | --- | --- | --- | --- |
| `registry-deps` | the workspace takes `markup5ever` 0.40.0, `tendril` 0.5.1 and `web_atoms` 0.3.0 from crates.io instead of its path members, so a git dependency on `html5ever` builds no other package from this repository | none | fork: the whole workspace's `cargo test` (its members build against the published crates); Viper: `viper-html::html5lib tree_construction_subset` | MIT OR Apache-2.0 (Viper's own change) | Viper-only; never offered |
| `svg-script-pause` | the tree builder pauses after an SVG `script` element's end tag, and after a self-closing SVG `script` start tag, as the HTML Standard's rules for parsing tokens in foreign content require, so the script is processed before parsing goes on | `S3-SVG-SCRIPTS` (Viper's register; closed by the Viper task that pins this patch) | fork: `html5ever/tests/viper_patches.rs` `an_svg_script_end_tag_pauses_the_parser`, `a_self_closing_svg_script_pauses_the_parser`, `other_script_end_tags_in_foreign_content_do_not_pause`; Viper: `viper-html::parser an_svg_script_pauses_the_parser_at_its_end_tag` | MIT OR Apache-2.0 (Viper's own change, written from the HTML Standard) | candidate for upstream issue #118; never offered without the owner |

### `registry-deps`

`Cargo.toml` (the workspace root): `[workspace.dependencies]` names
`tendril = "=0.5.1"`, `web_atoms = "=0.3.0"` and `markup5ever = "=0.40.0"`
without a path. Every member that takes them through `workspace = true`
(`html5ever`, `xml5ever`, `markup5ever_rcdom`, the `markup5ever` member
itself) builds against the published crates, so the workspace's types stay
consistent and its tests run unchanged. The path members `markup5ever`,
`tendril` and `web_atoms` stay in the workspace and are not used by
`html5ever`.

### `svg-script-pause`

HTML Standard, "The rules for parsing tokens in foreign content"
(<https://html.spec.whatwg.org/multipage/parsing.html#parsing-main-inforeign>):

- **An end tag whose tag name is "script", if the current node is an SVG
  `script` element:** pop the current node and pause the parser so the SVG
  script element is processed, then go on. `rules.rs` (`step_foreign`) now
  matches this case before "any other end tag" and returns
  `ProcessResult::Script` with the popped element, as the "text" insertion
  mode does for an HTML script's end tag. The insertion point and the
  script nesting level belong to the caller (the parser driver), as for an
  HTML script; html5ever's `TokenizerResult::Script` hands it the element.
- **A start tag with its self-closing flag set whose tag name is "script"
  and whose new current node is in the SVG namespace:** acknowledge the
  flag and act as for that end tag. `mod.rs` (`foreign_start_tag`) inserts
  and pushes the element, pops it and returns `ProcessResult::Script`
  (a returned script acknowledges the flag: no "Unacknowledged
  self-closing tag" error). A self-closing MathML `script` keeps the
  generic rule (inserted without being pushed, no pause).
- A "script" end tag whose current node is another element (for example
  `<svg><script><g></script>`) is "any other end tag", as before: no pause.

Trees are unchanged: the patch changes when the tree builder returns, not
what it builds, so the html5lib tree-construction tests pass as before.

## Rebase procedure (fork side)

Viper's ADR 0035 holds the whole procedure; these are its fork-side steps.

1. **Trigger.** A new html5ever release that Viper adopts; a soundness fix
   may trigger one earlier.
2. **Verify the base**: each published `.crate` SHA-256 against the
   crates.io index, its `.cargo_vcs_info.json` naming its commit, and its
   unpacked sources compared with the new base, apart from the files
   generated at packaging time. Record the result in the "Base" section,
   and decide again which packages `registry-deps` takes from crates.io.
3. **Create the branch** `viper/html5ever-<version>` at that commit and
   cherry-pick the patch commits in ledger order, keeping the markers. A
   patch that upstream made unnecessary is retired: its row moves to a
   "Retired" table with the upstream commit, and its Viper proof must
   still pass without it. A patch that no longer applies is reworked as a
   new commit with the same id.
4. **Test.** `cargo test --workspace` in the fork and each patch's fork
   tests. Update this file: `Base:`, `Branch:`, the base, the ledger and
   the rebase log, with conflicts and effort in hours.
5. **Review.** An independent review of
   `git range-diff <old base>..<old pin> <new base>..<new pin>`.
6. **Push the new branch.** Never force-push; never delete a branch that
   holds a commit any Viper revision pins.

## Rebase log

| Date | From → to | Patches carried / retired | Conflicts | Effort (hours) | Reviewer | Viper PR |
| --- | --- | --- | --- | --- | --- | --- |

## Pin log

Each Viper revision that moves its pin to this branch.

| Viper task | Pinned commit | Change |
| --- | --- | --- |
| V8-5 | the last commit of the range that adds this row | `registry-deps`, `svg-script-pause` |
