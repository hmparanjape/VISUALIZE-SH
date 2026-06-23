# Scheduled update routine

This file holds the prompt for a periodic assistant run that keeps the data fresh.
Wire it up with `/schedule` (or any Claude Code scheduled task) at whatever cadence
you like (e.g. monthly). The routine only ever produces **drafts**; you review and
promote them.

## How the loop works

1. The routine reads `schema/DATA_DICTIONARY.md` and the existing `data/*.yaml`.
2. It researches recent developments and proposes additions/edits as `draft`
   entities with `sources`.
3. It runs `npm run build:data` and fixes any validation errors.
4. It leaves the changes (ideally on a branch / as a diff) and a summary.
5. **You** review drafts in the app (dashed nodes / "Show drafts" filter),
   correct them, and flip `curation.status` to `curated`, then redeploy.

## Prompt to schedule

> You are updating the VISUALIZE-HF dataset. Work only inside `data/*.yaml`.
>
> 1. Read `schema/DATA_DICTIONARY.md` and follow it exactly, including the
>    "Hard rules for an automated update routine".
> 2. Review the current `data/*.yaml` so you don't duplicate existing entities.
> 3. Research developments since the most recent `curation.lastUpdated` in the
>    structural heart space (valvular and non-valvular): new FDA/CE approvals,
>    pivotal trial readouts, new devices/drugs/digital therapeutics, company
>    acquisitions, and status changes (e.g. investigational → approved).
>    Prioritize: LAAO, septal & congenital defect closure, HCM (myosin inhibitors
>    + septal reduction), ATTR cardiac amyloidosis, HFrEF/HFpEF pharmacotherapy,
>    interatrial shunts, implantable PA/IVC/LA sensors, coronary-sinus / CMD
>    therapies, digital therapeutics, and valvular therapy — TAVR (incl. aortic
>    regurgitation), mitral TEER/TMVR/annuloplasty, tricuspid TEER/TTVR,
>    transcatheter pulmonary valves, and surgical valve therapy.
> 4. Add or update entities as `curation.status: draft` with `curation.lastUpdated`
>    set to today and at least one credible `sources` URL each. Create referenced
>    companies/conditions before the entities that point to them. Reuse existing
>    `category`/`subtype` vocabulary and id prefixes.
> 5. Set/refresh the `pulse` field (0–10) to reflect *current* news attention:
>    score new entities, and re-score existing ones up or down as the cycle moves
>    (a quiet topic should drift down). `pulse` is the one field you may change on a
>    `curated` entity without flipping it to draft — it is a presentation signal,
>    not a clinical claim. Keep scores relative across the whole dataset.
> 6. Do NOT modify the other facts of existing `curated` entities. If something
>    curated looks outdated, instead add a `curation.notes` flag on it describing
>    the suggested change for human review.
> 7. Run `npm run build:data` and fix every error until it passes.
> 8. Output a concise summary: each id you added/changed, what it is, the source,
>    and anything the human curator should double-check (especially regulatory
>    status and NCT ids).
>
> Accuracy over completeness — when uncertain, omit optional fields (especially
> `nctId`) rather than guessing.
