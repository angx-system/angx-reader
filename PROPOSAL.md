# Proposal

*A potential stack for angx-reader.*

---

angx-reader runs alongside an ANGX client, on whatever device that client
already runs on — a steward's laptop or a base's persistent device. It reads
feeds the client has already replicated. It does not participate in
replication, transport, or notification over the network — those belong to
ANGX and to whatever bridge carries its feeds. Reader is a convenience layer
on top of the system, not a dependency of it — a steward or base can
register, log, and witness with only the client, and never run reader at
all.

---

## Two components, one package

angx-reader is made of two parts, always installed together.

**The indexer** — a silent, always-on background process. Set up once, on
first install, pointed at the local `feeds/` directory the client already
writes to. From that point it starts on boot and needs no re-activation. It
requires no network access of any kind — only local file access. Every time
a new signal lands in `feeds/`, the indexer embeds it and stores the
resulting vector in `vectors/`, continuously, without anyone opening
anything.

**The GUI** — a separate, minimal interface, opened on demand to view
matches. It is not part of the ANGX client and does not surface inside it.
When the indexer finds a new match, it triggers an OS-level desktop
notification; clicking it opens the GUI to the full match detail. The
steward does not need the GUI open for matching to happen — only to see
what's been found.

---

## 1. Source

angeliaX feeds, already replicated locally by the client, in `feeds/`. The
indexer watches this directory for new signals. It opens nothing over the
network itself; it reads what Hypercore has already written to disk.

`feeds/` requires no particular internal structure for reader's purposes.
Reader does not define or require folder-level organization by signal
type — filtering happens at the data level, not the filesystem level. Each
signal's own `Signal Type` field is what reader reads to decide whether to
embed it.

## 2. Embedding

`paraphrase-multilingual-MiniLM-L12-v2`, or equivalent. Multilingual
embedding model. Runs on CPU via ONNX Runtime. Downloaded once at setup.
Converts signal text into a vector — two signals about the same problem
land near each other regardless of language, with no translation layer.
Zero internet, zero cost per query.

## 3. Matching

Only `failure` and `learning` type signals are embedded and matched.
`operational` and `retired` signals are never vectorized — there is no
meaningful pairing for either. This applies equally to steward signals and
witness signals: a witness's `learning` signal, logged after directly
replicating a method, is as valid a match target as the original steward's
own.

A `failure` signal matches only against `learning` signals — documented
fixes, methods, hacks, confirmed solutions. This applies across both
operational and commons feeds: a commons failure (e.g., crop heat damage)
can match an operational learning signal (e.g., a shade-structure method),
and vice versa. The type filter is hardcoded: `failure` → `learning` only.

No distance filter is applied to physical provisions. Location is one
signal among many a steward reads before acting, not a precondition for a
match to surface — a method proven in a matching climate can be more
relevant a continent away than an untested one next door.

Matches are returned as a **ranked list**, ordered by semantic proximity —
never a single best match. Two thresholds apply: a minimum relevance score
below which a signal is not returned at all — a failure with no genuinely
close learning signal should return an empty list, not a list of loosely
related noise — and a maximum result count, so the steward reads a short,
ranked set rather than every signal that scored above the floor. Exact
values are a tuning decision for implementation, not fixed here.

The match list is never static. It is re-derived live from the current
vector index. A new learning signal arriving anywhere in the collection can
surface a new match for an old failure without anyone re-querying manually
— the indexer is always running, so the next time a lookup happens, the
result reflects whatever is currently true.

Matching never weighs outcome history. A learning signal with
contradicting witness failures ranks identically to one with only
corroborations — reader surfaces both signals; judging what the
contradiction means is left entirely to the steward reading them.

## 4. Reader on a base

Everything above applies identically whether reader runs on a steward's own
device or on a base's persistent device (a base's Raspberry Pi, for
example). A base's `feeds/` holds full local copies of every node it
curates, per the Collection Log — the indexer treats this exactly like any
other `feeds/` directory, vectorizing every `failure` and `learning` signal
across the entire collection, automatically, with no exception.

A base is not a subject seeking matches for itself. It holds a library of
vectors on behalf of the stewards whose work it curates, the same posture
it already holds toward the feeds themselves — copies, not ownership
(Constraint 5). Because a base's collection is typically far larger than
any individual steward's own replicated feeds, a base running reader is the
primary, load-bearing source of useful matches across the network — not an
edge case.

## 5. Getting matches — three paths

**Primary — base-hosted match lookup by Node ID.** If a steward's node is
curated by a base, that base is already vectorizing its entire collection,
continuously. The steward queries the base (same p2p-address pattern as any
manual query) and supplies their own Node ID; the base returns its current,
already-computed ranked list of matches for that node. No local reader
required, no live computation triggered — a lookup against an index the
base has already kept current.

Operational tab → [enter base p2p address, or select an already-partnered base]
→ Check Matches → [enter own Node ID]

This query happens in the **client**, not in reader's own GUI, over the
network the client already uses for everything else — replicating feeds,
querying a base, all of it. Reader itself opens no connection here; it only
computes the answer locally, on the base's own device, from data already
sitting in its own `feeds/`.

**Fallback, for any node, curated or not — manual text/field query.** The
same mechanism already used for keyword search ("pythium" against a base's
food nodes) — querying a base by signal type, location, text, or any other
indexed field. This works whether or not the querying steward's own node is
held by that base, since it searches the base's Hyperbee index directly,
not its computed match index.

**Optional, local — a steward's own reader instance.** Worth running when a
steward holds work not curated by any base, or wants matching independent
of any base's collection. Not the default path for most stewards; a
supplement for niche work or work unreplicated elsewhere.

## 6. Provenance — lineage clustering

Nodes may optionally declare a `built from` reference at registration —
an external source (a URL) or a Node ID found through ANGX itself. The
indexer clusters nodes that cite the *same* `built from` value as
**siblings** — grouped together regardless of how different their own
descriptions read, since the grouping is on the field value, not
semantic content. This is a second, separate matching mode from
failure → learning — it answers "who else built from this," not "what
solves this."

**Example.** A steward in Blantyre posts a gravity-fed chlorine doser,
citing a public GitHub repo as `built from` (an external reference). A
steward in the Philippines finds this node through ANGX and builds their
own, citing the Blantyre node's **Node ID** as `built from`. A third
steward, in an unrelated region, independently finds the same Blantyre
node and does the same. The Philippine and third steward's nodes are
**siblings** — both cite the same Node ID — and the indexer clusters
them together, discoverable from either one: reading either sibling's
entry surfaces the other, along with the shared `built from` reference
they both point to. The Blantyre node itself is not a sibling of either
— it's what they both point to, not something pointing anywhere itself.

Clustering applies equally to external references: two or more nodes
citing the same external source as `built from` cluster as siblings the
same way.

## 7. Output

A match is a pointer to two existing signals, nothing more. Reader writes
nothing to the protocol and has no authority over it. The steward reads and
decides what to do.

---

## What this proposal does not include

Reader contains no networking code in this version. It reads local files
and answers lookups from data already on the same device — nothing more. A
steward wanting matches from data reader doesn't already hold locally must
query and replicate the relevant feeds through the client first; the
indexer picks up new signals automatically once they land in `feeds/`.

---

## Open Questions

- **Base-hosted match lookup — query interface.** Section 5's primary path
  assumes a base exposes its computed match index as a read-only lookup by
  Node ID. The interface itself isn't yet defined.
  
- **Push vs. pull.** This proposal is pull-only. Should a base ever push a
  relevant match to the steward it concerns, rather than waiting to be
  asked? If so, that belongs in the client, which already handles incoming
  network messages — not in reader. Undecided.
  
- **Remote vector matching.** A steward's own vectors checked live against
  a remote base's, without replicating first. Requires reader to expose a
  network service it doesn't have here, and raises a real question — who's
  allowed to query. Not part of this proposal.
  

---

Every component is open source, self-hostable, and replaceable. No vendor
dependency. The right implementation of any layer may look different from
what is proposed here.

---

*angx-reader*
