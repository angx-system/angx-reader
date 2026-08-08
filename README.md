# angx-reader

*A lens over angeliaX.*

---

angeliaX records the operational reality of work and surplus — work built,
methods learned, failures encountered, standing commons provisions. At
sufficient density the connections between those records become impossible
to find by reading alone. A failure logged in one place and its solution
logged in another may never meet.

angx-reader finds those connections. It writes nothing. It has no authority
over the protocol. It surfaces matches — the steward decides what to do
with them.

---

## The boundary

angeliaX does not record what is needed, requested, or wanted. That line is
deliberate and angx-reader does not cross it.

The reader pairs a failure signal only with learning signals — documented
methods, fixes, hacks, confirmed solutions. It does not matter which log
the failure or the learning signal belongs to. A commons failure (e.g.,
crop heat damage) can be solved by an operational learning signal (e.g., a
method for building a shade structure). Both steward and witness signals
are matched — a witness's learning signal, logged after directly
replicating a method, is as valid a match target as the original steward's
own.

Because the filter is failure → learning only, a failure can never surface
a commons node's surplus either — surplus is never logged as a learning
signal — it's stated once at registration and confirmed through ongoing
`operational`-type signals on the commons node itself, so it was never a
valid match target to begin with.

**Example.** A food bank in Lima logs a failure signal: "50kg rice spoiled
this week. Storage humidity too high, no dehumidifying method in place."
The reader searches only for learning signals — and finds one, logged eight
months earlier by a farm on another continent: a low-cost humidity-control
method for grain storage, documented in full. The steward reads it, tries
it, and logs the outcome on their own node. Nothing was requested. Nothing
was matched to a need. A failure met a method that already existed.

---

## Where matching actually happens

Reader runs quietly in the background wherever it's installed — on a
steward's own device or on a base's persistent device — continuously
embedding every `failure` and `learning` signal it finds in the local feed
store.

**Bases are where this matters most.** A base curates and replicates a full
collection of nodes — often far more than any individual steward holds on
their own. Reader running on a base's device vectorizes that entire
collection automatically, computing matches across every node it holds, on
an ongoing basis, whether or not anyone asks. The base isn't matching for
itself — it's holding a continuously updated library of matches on behalf
of everyone whose work it curates.

**For most stewards, this means running reader locally is optional.** If a
node is curated by a base, that base has already done the work. The steward
queries the base directly, by their own Node ID, and reads back whatever
the base's reader has already found — current as of the last signal that
landed anywhere in that collection. Running a local reader instance still
has a place — for work not yet curated anywhere, or for matching
independent of any single base's collection — but it isn't the primary
path.

Anyone can also query a base by keyword or field — signal text, node type,
location — whether or not their own node is held there. That path doesn't
need reader at all; it searches the base's own index directly, the same way
any other query works.

Both kinds of query against a base — by Node ID, or by keyword — happen in
the ANGX client itself, initiated by the steward, over the network the
client already uses for everything else. Reader's own GUI shows matches a
steward's own local instance has found on their own device; it does not
currently receive anything pushed from elsewhere.

---

## What it finds

Each signal becomes a point in a vector space — close to other signals
about the same thing, far from the rest, regardless of language.

A failure signal that lands near a learning signal points to a problem
already solved somewhere else in the record. A method that lands near a
problem points to expertise logged by someone who never heard of the
person who needed it.

Nodes that cite the same `built from` reference — the same external design,
or the same upstream node — can be grouped as a family, even if their own
descriptions read nothing alike.

Matches are returned as a ranked list, ordered by semantic proximity —
never a single best match. A minimum relevance threshold keeps unrelated
signals out: a failure with no genuinely close learning signal returns an
empty list, not a page of loosely related noise. A maximum result count
keeps the list short and readable rather than exhaustive. The list is never
static — it's re-derived live from the current vector index, so a new
learning signal logged anywhere in the collection can surface a new match
for an old failure without anyone re-querying. None of it moves anything.
Each result is a pointer to two places in the existing record. The steward
reads and decides.

---

## How a match closes the loop

A match isn't the end of the story — applying it and logging the outcome is
what makes the record stronger for the next person.

When a steward finds a learning signal that solves their failure, they apply
it, log their own outcome as a new signal on their own node, and post a
witness signal on the learning signal they tried. If it worked, that witness
signal is a `learning`-type entry — it enters the vector space too, adding
corroboration to the original method. If it didn't — wrong climate, wrong
materials, or the method simply doesn't hold up — the witness signal is
`failure`-type instead, sitting permanently alongside the original learning
signal, visible to the next steward evaluating the same match. Neither
outcome is hidden. The next steward hitting the same failure, anywhere in
the network, finds a solution with either more confirmation behind it than
the last person had, or an honest record of where it didn't hold.

---

## Docs

- PROPOSAL.md — the proposed stack

**Dependency:** [github.com/angx-system/angeliax](https://github.com/angx-system/angeliax)

---

*angx-reader*
