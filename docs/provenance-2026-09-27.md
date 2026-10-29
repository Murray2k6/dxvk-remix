# Code provenance check against doodlum/dxvk-remix, 2026-09-27

Result: the only code in this tree taken from doodlum/dxvk-remix is the credited
retained-instances port (commit `caaa21b47`, from their `5e39a218a`). Everything else
added on `rebased-9-19-opt` since it branched is our own work.

## What was compared

| | Range | Meaningful added lines |
| --- | --- | --- |
| Ours | `681c6de9a` (branch point from `rebased-9-19`) .. `rebased-9-19-opt` | ~15,300 |
| doodlum | `0853746a5` (their merge base) .. `doodlum/skyrim` `ab36236a1` (tip on 2026-09-27, 17 commits) | ~4,550 |

"Meaningful" = at least 30 characters, not a comment or `#include`. The remote tip was
re-fetched on 2026-09-27 and had not changed (`git ls-remote`: `refs/heads/skyrim` =
`ab36236a18beaf9131028a3708bcbea40383e4bd`).

Authorship: all 171 non-merge commits in our range are by Kim2091.

## Method and results

1. **Exact line match.** Whitespace-normalised added lines of ours that equal an added line of
   theirs: 180. After removing lines that already existed anywhere in the base tree
   (`681c6de9a`; boilerplate such as lock guards, return codes, loop headers): **75**.
   - ~50: the retained-instances port (`SceneManager::setRetainedExternalDraw` /
     `setRetainedExternalDrawTransform` / `removeRetainedExternalDraw`, the retained table
     and index members, scene-origin rebase, replay loop) in `rtx_scene_manager.cpp/.h` and
     the retained entry points in `rtx_remix_api.cpp`. Credited in the commit message and in
     code comments ("after doodlum/dxvk-remix 5e39a218a").
   - ~5: the same compile fixes any port of DXVK's D3D11 frontend to this codebase needs
     (`vulkan11Features.shaderDrawParameters`, `SubmitPresent(..., imageIndex)`,
     `createRtxContext()` on the immediate context, a `.def` export). Dictated by the core API.
   - Rest: generic idioms (`for (uint32_t i = 0; i < count; ++i)`, `steady_clock::now()`).
2. **Renamed-copy check.** 24-token windows with every identifier replaced by a placeholder:
   1.5% of our windows also occur in their added code. The per-file peaks are shared
   boilerplate shapes (RtxOption declarations, option getters, timing blocks), not copied logic.
3. **Outlier check.** The DLFG pacing probe (`rtx_dlfg_probe.*`) showed the highest structural
   share (35%). Exact match against their DLFG/Reflex changes: **0 of 408 lines**, and none of
   their `CS_REMIX_*` environment gates. It followed up their published findings (credited in
   `CREDITS.md`) and has since been removed.

## Design difference (D3D11 host)

- Ours: DXVK's D3D11/DXGI frontends rebuilt against this codebase (`856e0f7ad`), all
  frontends in one module sharing one Vulkan device with Remix (`03fae340d`, `2d8f1e92f`), the
  API hosted on the game's own device (`9b749bbe1`: `remixapi_RegisterD3D11Device`,
  `remixapi_RenderD3D11`).
- Theirs: a separate API-only D3D11 stack (`dxvk_d3d11` / `dxvk_dxgi`) in its own
  `src/d3d11/d3d11_remix_api.cpp` (~1,750 lines). That file does not exist in this tree.

## Reproducing

```
git fetch doodlum skyrim
git diff 0853746a5 doodlum/skyrim > dood.diff
git diff 681c6de9a rebased-9-19-opt > ours.diff
```
Then compare the `+` lines of both diffs (normalise whitespace, drop lines present in
`git grep -h "" 681c6de9a`). The scripts used are simple enough to rewrite from this description.
