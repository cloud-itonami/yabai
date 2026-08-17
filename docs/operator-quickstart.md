# Operator quickstart

This repository holds **two corpora about real named third parties, built five months
apart under different standards of proof.** Neither is a target list and neither
should be read as one, but they do not make claims the same way, and a reader who
takes the older one's confidence into the newer one — or the reverse — will be wrong
in both directions.

It also contains **ten fully addressed abuse-report emails**, which is the fact most
worth knowing before touching anything here (§4).

`CLAUDE.md` opens by declaring itself deprecated: the actor moved to
`20-actors/yabai/actor-manifest.jsonld` (T1 MCP-Compose), and the `appview/` here is
"retained as T3 fallback only". So this repository is a corpus and a fallback, not
the live path.

Steps marked ✅ were run against this tree on 2026-08-16. One is marked NOT WALKED
with the reason.

| | `content/` | `origins/` |
|---|---|---|
| what | AML / sanctions entity risk | email sender domains |
| scored | 2026-03-16, one batch | as-of 2026-08-07 |
| size | 469 entities, 464 evidence | 174 domains, 1,151 messages |
| findings | 7 entities in the Deny band | Gmail corpus: **0 phishing, 0 abusive, 0 suspected**; owner report: **12 abusive domains + 1 abusive sender / 22 deliveries, 2 suspected domains / 3 messages** |
| retractions | none recorded | **5 false positives recorded** |

---

## 1. `origins/` says what it will not conclude, and the data obeys it ✅

`origins/README.md` makes three claims. All three hold.

**The declared counts match the rows.** `origins.edn` carries a `:counts` block —
a self-report, which is exactly the kind of thing that drifts:

```clojure
;; declared
{:domains 174, :messages 1151, :classified 70, :self-registered 43}
;; recomputed from :origins
{:domains 174, :messages 1151, :classified 70, :self-registered 43}
```

**Trust moved only where evidence exists.** 43 of 174 domains sit at
`:self-registered`, 131 at `:unverified`, and **all 43 have a non-empty
`:trust/evidence`** — zero exceptions. The relation is exact in both directions:
every `:self-registered` domain has `:origin/via-relay? true`, and **no `:unverified`
domain does.** That is the README's point that the one computable piece of evidence
is delivery through Apple's private relay, and the level means precisely that and
nothing more.

**Volume did not move trust.** The busiest `:unverified` domain has 26 observed
messages; `:self-registered` domains range from 1 to 522. The ranges overlap, so a
domain with 26 messages behind it is still recorded as unverified. The README says
"400 messages from a domain say nothing about whether it is who it claims to be";
the data does not quietly contradict it.

104 of 174 domains are `:origin/kind :unknown`, which the `:counts` block states as
`:classified 70`. Most of this corpus is unclassified and says so.

## 2. `abuse.edn` keeps the measured zero separate from reported spam ✅

The abuse ledger found nothing, and that is not the interesting part:

```clojure
:status {:corpus {:messages 1151 :domains 174 :as-of "2026-08-07" …}
         :authentication {:evaluated 1075 :authenticated 1039 :unaligned 36
                          :impersonation-suspected 0 :unknown 76}
         :phishing-found 0 :abusive-found 0 :suspected-found 0}
```

Four verdict kinds (`:phishing :abusive :suspected :legitimate`) and six evidence
kinds (`:auth-results :envelope-mismatch :ioc-match :link-mismatch :reporter` and
`:provider-classification`) are defined. The Gmail corpus still has no abuse finding,
while `:entries` now holds **twelve domain-level and one sender-level abusive verdicts
covering 22 spam deliveries**, plus two domain-level suspected-phishing verdicts
covering three messages. They were reported separately from a Microsoft 365 quarantine
deployment on 2026-08-17. `:false-positives` holds **five** records.

The reported entries publish only the sender address, lure subject, observation time,
provider classification and owner approval. Recipient identity, tenant data, internal
message IDs and quarantine-release URLs are deliberately absent. The report is not
folded into the Gmail corpus counters, so the corpus zero remains a truthful scoped
measurement rather than being overwritten by data from another deployment.

Count what is there, not what a key is named: `:verdicts` is a *map of four verdict
kinds*, not four verdicts issued. The verdicts actually issued are the fifteen entries;
reading the vocabulary itself as four more verdicts would still be wrong.

And the file states why the zero is not evidence of safety:

```clojure
:gaps [{:gap :no-link-extraction
        :blocks #{:link-mismatch}
        :note "本文中のリンクを抽出・比較していない。表示 URL と実リンク先の不一致は未検出"}]
```

One of the five evidence kinds **cannot be produced at all**, and the file names
which one. A phishing count of zero from a detector that never compares a displayed
URL against its target is a count of what was looked for, not of what is there. This
is the right way to publish a zero, and it is the first thing to read before quoting
`:phishing-found 0` anywhere.

## 3. `content/` holds together, and its no-information floor is distinguishable ✅

```bash
comm -3 <(ls content/entity | sort) <(ls content/risk | sort)   # empty: exact 1:1
```

469 entities, 469 risk records, a perfect 1:1, and 1,411 content files parse with
none unreadable. Evidence joins by `entityId`: 464 files naming 463 distinct
entities, **no evidence pointing at a missing entity**, one entity with two.

That leaves **6 entities with no evidence file**, and their risk records are
byte-identical to each other:

```clojure
{:wellBecomingScore 72 :penaltyScore 5 :yabaiRiskScore 32 :infoRisk 0}
```

This is the no-information floor, and it is unambiguous: of the 463 entities that do
have evidence, **zero** carry that tuple, zero have `infoRisk 0`, and zero score
exactly 32. So "nothing is known" cannot be mistaken for "assessed and low" —
the failure that makes a scored corpus dangerous. Assessed scores run 18.7 to 100.0.

The `@id` of every addressable record matches its filename across all 1,406
entity / risk / evidence / public records. The 5 files under `content/post/` do not,
because they are `app.bsky.feed.post` records and carry no `@id` at all — a
different shape, not a broken one.

**The graduated thresholds are not graduated on this data.** `CLAUDE.md` defines
Monitor ≥70, Challenge ≥85, Deny ≥95. Measured: 8 entities are ≥70, 8 are ≥85, and
7 are ≥95. The Monitor band (70–85) is **empty** and the Challenge band holds exactly
one. Seven of the eight flagged entities land straight in Deny. Nothing is wrong with
the arithmetic; the escalation ladder simply has no rungs in the middle on this
snapshot, and anyone relying on "Monitor first" should know that.

All 469 records share two `scoredAt` values. This is one batch from 2026-03-16, not a
live feed — and the model applies recency decay (≤30d → 1.0, ≤1y → 0.9), so the same
evidence recomputed today would not produce these numbers.

## 4. ⚠ Ten addressed, ready-to-send abuse emails are committed here

`tools/track-phishing-infra/abuse-drafts/` contains ten `.eml` files with complete
headers, addressed to real abuse desks:

```bash
grep -h '^To:' tools/track-phishing-infra/abuse-drafts/*.eml | sort -u | wc -l
#   9   ← from 10 files
```

Nine recipients from ten files: **two drafts go to the same desk**
(`abuse@dynadot.com`, once as "Dynadot Inc" claiming 51 domains and once as
"Dynadot LLC" claiming 23). The split is by registrar name string, and those two
strings are plausibly one company. Sending both sends one desk two reports about a
possibly overlapping set.

Every `Date:` header is `19 Apr 2026 11:22:50` or `:51` — a single generator run,
four months old. The subjects claim 118 domains across six registrars and 147 across
four ASNs.

**Nothing in this repository sends them.** They are artifacts of one batch run.
Contacting a third party is an outbound action and belongs to the actor's governed
path, not to a directory of files — and a four-month-old domain list is not what you
would send even if it were.

## 5. ⚠ The drafts' claims cannot be checked from a checkout

`generate-drafts.mjs` reads its domain lists from a database, not from this repo:

```javascript
const PG = ["-h", process.env.RW_HOST ?? "45.32.79.245", "-p", process.env.RW_PORT ?? "4566", …]
const FROM     = process.env.ABUSE_FROM     ?? "abuse-liaison@etzhayyim.com";
const REPLY_TO = process.env.ABUSE_REPLY_TO ?? "jun@etzhayyim.com";
```

So the "51 phishing domains" in a subject line is not verifiable here — the input is
an external RisingWave instance at a hardcoded default host, and the emails are the
only surviving artifact of what it returned.

The committed drafts also **do not match those defaults**: they carry
`From: abuse-liaison@etzhayyim.ai` and `Reply-To: jun@etzhayyim.group`. They were
generated with environment overrides, so regenerating without the same environment
produces different headers on otherwise similar mail.

(`generate-drafts.mjs` is a `.mjs`, which the workspace forbids for new files —
those go in nbb. This one predates the rule; it is noted, not changed.)

## 6. The TypeScript tests ⚠ NOT WALKED

`kotoba/` has a vitest suite against `src/index.ts` through `@etzhayyim/sdk-mock`,
which would be the cheapest real gate in the repo. It does not install here:

```bash
node <root>/scripts/resource-guard.mjs run build -- npm install --no-audit --no-fund
#   npm error code EALLOWSCRIPTS
#   npm error --allow-scripts is not allowed in project-scoped installs.
#   Add the entries to the "allowScripts" field in package.json, or to .npmrc, instead.
```

Both dependencies are git URLs (`@etzhayyim/sdk`, `@etzhayyim/sdk-mock`) whose
install wants to run scripts, and npm 11.16.0 on this machine refuses that in a
project-scoped install. This is a local-toolchain limit, not a repository defect —
the same class as `npx --yes` misparsing flags here while working on the fleet nodes.
So the suite is not claimed to pass, and `tsc --noEmit` was not run either.

## 7. What the maturity instrument cannot see here ✅

```
· orgs/cloud-itonami/yabai  own=0.043
    axis-ingest は 0bp だが計数外のファイルに URL が 22 件在る
    README が .md ではないので docs の README 成分は 0（README.edn 等が 1 件）
```

1,461 tracked files, two corpora, a runbook, a risk-design doc and an accepted
enforcement ADR, scored 0.043. The README component reads `README.md` and this
repository's is `README.edn`; the citation counter looks only under
`facts`/`catalog`/`data`, so `content/` and `origins/` are invisible to it. Recorded
in ADR-2608052000 as blind spots, not absences.
