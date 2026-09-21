# NPC / Konekt Modernization — Meeting Summary

## Context and attendees

The call included Kevin Lim, GokulKumar Ravichandran, Helan Magdalin Thomas, Brian Roster, and Nathan Sellers (running transcription). Kevin opened by confirming that a separate topic someone raised wasn't on today's agenda and would be handled in another meeting.

## Agenda framing

GokulKumar set the agenda: the team is running two parallel modernization tracks — the NPC cloud modernization and the Konekt re-platforming from Azure to AWS. He described NPC as the larger-scale effort, since it involves retiring major Windows-level dependencies and replacing them with AWS-native, cloud-native components.

## Search modernization — the main technical discussion

GokulKumar walked through where things stand on the search/indexing piece of NPC:

- **Current state:** NPC uses MySQL full-text search paired with a compiled C word-breaker plugin, which handles the indexing.
- **Proposed future state** (carried over from a prior day's discussion, being recapped here for the group): move to Amazon OpenSearch with a custom analyzer built to replace the word-breaker plugin's functionality, plus a full migration of the existing MySQL full-text indexes into OpenSearch.
- This OpenSearch-with-custom-analyzer approach is the one concrete decision reflected in the transcript — it was already proposed and is being revisited here rather than newly agreed in this session. The transcript doesn't capture the rest of GokulKumar's walkthrough of "yesterday's meeting" discussion in detail (the excerpt jumps ahead), so there may be additional nuance from that session not captured here.

## Email notification impact — new concern raised

Brian Roster raised a consideration that hadn't been addressed in the original proposal: NPCS's current email notification flow is file transfer → connect to an SMTP/SMB server → send notification. Many customers point NPCS at their own on-premise mail server rather than Office 365. His concern: once NPCS runs in the cloud (potentially using SES), a customer who wants to fully decommission their on-prem NPC infrastructure would have no on-prem mail server left for NPCS to relay through — creating a functional gap for that customer segment.

GokulKumar's initial reaction was that he didn't expect a major impact, reasoning that email delivery isn't purely dependent on on-prem infrastructure and that AWS has native options (SES) available — but he stopped short of a firm conclusion and agreed he needed to review the options more closely rather than assume no impact.

Brian noted this was just one of potentially several considerations he'd flag after reading the initial proposal, implying there may be more gaps to surface later.

## Closing and process items

- Helan reminded the group that action items need to be tracked on the Confluence ("conference") page and shared with the students/team.
- Kevin agreed, offered to help facilitate any further questions the team has for GokulKumar, and confirmed more follow-up working sessions need to be scheduled.
- GokulKumar confirmed he had no outstanding questions for this session, said he wanted to "reverse" (review) the material once more, and would set up the next round of working session meetings himself.

## Action items

| # | Owner | Action |
|---|-------|--------|
| 1 | GokulKumar Ravichandran | Review whether moving NPCS to the cloud (SES vs. on-prem SMTP relay) creates an impact for customers who currently rely on their own on-prem mail server for NPCS notifications; report back with findings. |
| 2 | GokulKumar Ravichandran | Re-review the full modernization proposal himself, then schedule/organize the next working session(s) to continue the NPC and Konekt modernization discussions. |
| 3 | Team (Helan / GokulKumar) | Log this meeting's action items on the Confluence page and share with the broader team. |
| 4 | Kevin Lim | Remain available to help facilitate any additional questions the team surfaces for GokulKumar between now and the next session. |
| 5 | Brian Roster (implied, not explicitly assigned) | Flagged that he may have additional concerns beyond the email notification issue after further reading of the initial proposal; worth following up with him directly if a fuller list is needed. |

## Gap in the transcript

This transcript excerpt skips from the ~1-minute mark straight to the 2-hour-11-minute mark, so the bulk of the middle discussion (roughly two hours) isn't included here — there may be additional decisions or action items in that unlabeled gap that aren't reflected in this summary.
