# Proposal

*A potential stack for angx-reader.*

---

angx-reader runs alongside an ANGX client, on whatever device that client already runs on — a steward's laptop or a base's persistent device. It reads feeds the client has already replicated. It does not participate in replication, transport, or notification — those belong to ANGX and to whatever bridge carries its feeds.

---

**1. Source** — angeliaX feeds, already replicated locally by the client. The reader watches `feeds/` for new signals. It opens nothing over the network itself; it reads what Hypercore has already written to disk.

**2. Embedding** — `paraphrase-multilingual-MiniLM-L12-v2`, or equivalent. Multilingual embedding model. Runs on CPU via ONNX Runtime. Downloaded once at setup. Converts signal text into a vector — two signals about the same problem land near each other regardless of language, with no translation layer. Zero internet, zero cost per query.

**3. Matching** — each new signal is passed to the embedding model and the resulting vector stored in `vectors/`. Matching is filtered by the signal's own type field, already explicit in the schema — vector proximity alone is never enough:

A `failure` signal matches only against `learning` signals — documented fixes, methods, hacks, confirmed solutions. This applies across both operational and commons feeds: a commons failure (e.g., crop heat damage) can match an operational learning signal (e.g., a shade-structure method), and vice versa. The type filter is hardcoded: `failure` → `learning` only.

No distance filter is applied to physical provisions. Location is one signal among many a steward reads before acting, not a precondition for a match to surface — a method proven in a matching climate can be more relevant a continent away than an untested one next door.

Matches are ordered by semantic proximity. The underlying record never changes.

**4. Provenance** — nodes may optionally declare a `built from` reference at registration — an external source or another node's ID. The reader can also cluster on this: nodes sharing the same `built from` value surface as siblings, independent of semantic proximity. This is a second, separate matching mode from failure → learning — it answers "who else built this," not "what solves this."

**5. Output** — a match is a pointer to two existing signals, nothing more. The reader hands matches to whatever notification path the local client already uses. It defines no transport or messaging protocol of its own.

---

Every component is open source, self-hostable, and replaceable. No vendor dependency. The right implementation of any layer may look different from what is proposed here.

---

*angx-reader*
