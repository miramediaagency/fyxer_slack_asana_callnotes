# Mira Media — Call Notes → Slack + Asana Automation

**What this does:** After a client call, pulls the Fyxer summary and action items, posts
takeaways to the right client Slack channel (internal-only channels), and creates an
Asana task per action item, assigned to the named owner.

**How to set this up (each team member does this individually):**
Fyxer is private per-user — nobody else's calls are visible to you, and yours aren't
visible to anyone else, including admins. So this can't be run centrally by one person
for the whole team. Everyone who wants their own client calls auto-processed needs to:

1. Open Claude Cowork → Scheduled tasks → New task
2. Connect your own Fyxer, Slack, and Asana accounts (if not already connected)
3. Set it to recur every **10–15 minutes** (this is a polling loop, not a push
   trigger — Fyxer has no "recording ready" webhook, so a call is only ever
   picked up on the next scheduled run; 10–15 min keeps that gap tight without
   over-polling)
4. Paste the instructions block below as the task prompt

Everyone uses the identical instructions — only your own connected accounts differ.

---

## Task instructions (paste as-is)

```
Every time this runs:

1. Call Fyxer find_recordings for calls completed since the last run.
2. For each new recording, call get_recording for the summary and action items.

3. Match the meeting title against this table (case-insensitive substring match on
   name, acronym/nickname, or known meeting-title pattern):

   | Client                          | Match on                                     | Slack channel | Asana project GID |
   |----------------------------------|-----------------------------------------------|---------------|--------------------|
   | Alcorn Pumps                     | Alcorn                                         | C07F8K8LBNC   | 1209814971271443   |
   | Anomalo                          | Anomalo                                        | C09NHUDV079   | 1211983160883467   |
   | Ascent Cloud                     | Ascent Cloud                                   | C0BFW97AAJK   | 1216539667890665   |
   | Atlantic Industries Limited      | AIL                                             | C085ENCTRU4   | 1209183793685490   |
   | Bloomwell                        | Bloomwell                                      | C0AS5J2LN3C   | 1214076786360041   |
   | Cheqroom                         | Cheqroom                                       | C09TY7EDEF8   | 1212023553029474   |
   | Clayton Park Audiology           | Clayton Park                                   | C058MLRR1QV   | 1206972531856910   |
   | Construction Safety NS           | CSNS, Construction Safety                      | C09PAF0BV5J   | 1211733794801895   |
   | Country Kitchens Online (CKO)    | CKO, Country Kitchens, CedarBrands             | C0A50D9DR8E   | 1212650283092399   |
   | CPA Atlantic                     | "CPA Atlantic" (do NOT match bare "CPA")       | C0778DPFE06   | 1207409326439161   |
   | Destination Cape Breton          | DCB, Destination Cape Breton                   | C080Q7HQ4TC   | 1208760629917551   |
   | Dot and Co                       | DOT, Dot and Co, Dot & Co                      | C085C4G37H9   | 1208982271817238   |
   | East Peak Climbing               | East Peak                                      | C0C0ABXK0MC   | 1218291334602820   |
   | Explore Waterloo                 | EWR, Explore Waterloo                          | C0880FUFCFJ   | 1208287148861606   |
   | Gentle Touch                     | Gentle Touch, GT (only if unambiguous)         | C069D6LF9M4   | 1206972531856904   |
   | Invest NS                        | INS, Invest NS                                 | C087295RL4B   | 1209194636659584   |
   | LiftLab                          | LiftLab, Lift Lab                              | C096HLA66N7   | 1211079709922858   |
   | MadeGood                         | MadeGood, MG, Riverside x Instacart, Accrue x Mira | C0AJ5E6SE7N | 1213552256590643 |
   | Mansfield Hall                   | Mansfield Hall                                 | C077HGQRRE0   | 1209814971271443   |
   | Mildren Plumbing                 | Mildren                                        | C08KKJ1FVDG   | 1209814971271443   |
   | Mount Saint Vincent University   | MSVU                                           | C078EQ5K2TV   | 1207550815232901   |
   | Nova Scotia Health Authority     | NSH, PACE, OHPR (NOT bare "Haylo" — see below) | C067HHYNA7R   | 1206972531856934   |
   | Pindrop                          | Pindrop                                        | C0A174RLZNZ   | 1212650283092512   |
   | Stewart McKelvey                 | Stewart McKelvey                               | C0AK357L9EX   | 1209194636659712   |
   | Tidal Hearing                    | Tidal Hearing                                  | C05QKG9BWVD   | 1206972531856904   |
   | Teamwork Cooperative             | Teamwork Cooperative, Autism at Work           | C0AKX8A44RY   | 1207238956620516   |
   | Trane                            | Trane                                          | C09V9T90LEM   | 1214591066676077   |
   | Wedgwood Insurance               | Wedgwood                                       | C08CS5RK53N   | 1209372963852357   |
   | Wellnest Fertility                | Wellnest                                       | C0BE65VDRK4   | 1216184383986065   |

   SPECIAL CASE — do NOT auto-route:
   If the title contains "Haylo" (e.g. "Mira x Haylo Weekly") without also naming a
   specific client, it's ambiguous — could be Nova Scotia Health, Atlantic Lottery,
   CPA Atlantic, or TeamWork Cooperative (all routed through Haylo Branding). Do not
   post to Slack or create Asana tasks automatically. Flag it for manual review with
   the full summary attached instead.

   If no match at all, skip Slack/Asana and note the unmatched title.

4. SLACK SAFETY CHECK — before posting to any matched Slack channel:
   - Call slack list channel members for that channel.
   - Build the "internal" allowlist as: any email on the @miramediaagency.com domain,
     PLUS any email on the @streamlinebusiness.ca domain (Mira's bookkeeping partner),
     PLUS this explicit addition: andreavazquezn1898@gmail.com (Andy — Mira Media team
     member using a personal email address).
   - If ANY channel member's email is NOT in that internal allowlist, DO NOT post to
     Slack. Log it as "skipped: external member present" and still proceed to step 6
     (Asana task creation is internal-only and unaffected by this check).
   - Only post to Slack if every member in the channel is in the internal allowlist.

5. For each action item, identify the named owner. Look them up in Asana's workspace
   users by name match, and create a task in the matched project, assigned to them,
   titled with the action item text. If no due date is mentioned, set one week out.
   If no matching Asana user is found, create the task unassigned and note the
   intended owner in the task description.

6. If the safety check (step 4) passed, post a Slack message to the matched channel
   with the meeting title and date, followed by two sections:
   - "Key takeaways" — the call's key takeaways as bullet points.
   - "To-do's (created in Asana)" — one bullet per action item created in step 5,
     formatted as "Owner — action item text (due date)".
   If the safety check failed, skip this step (the Asana tasks from step 5 still stand).
```

---

## Known gaps / decisions baked in

- **Slack posting will rarely fire.** Most of these are client-facing channels, so the
  safety check will usually skip the Slack post and only create the Asana tasks. This
  was a deliberate choice: any non-Mira/non-partner member in the channel blocks the post.
- **Haylo-routed clients** (Nova Scotia Health, Atlantic Lottery, CPA Atlantic, TeamWork
  Cooperative) fall back to manual review if the meeting title just says "Haylo" without
  naming the specific client.
- **Andy's personal email** (andreavazquezn1898@gmail.com) is the only non-agency-domain
  individual address treated as internal, in addition to the whole @streamlinebusiness.ca
  domain (Mira's bookkeeping partner, e.g. Lisa Gray). Add anyone else here if the team
  roster or partner list changes.
