# MEDIA_RESOLUTION_DISCLOSURE.md

Prior Art Disclosure — Memory-Grounded Media Resolution, Self-Utterance Playback Parsing, and Proactive Conversational Curation

Author: Oleg Rein (VIOLIRA), Ålgård, Norway
GitHub: github.com/violira
Public disclosure date: 11.07.2026
Implementation in production since: April 2026 (dated evidence below)
License: MIT. Purpose: defensive publication (prior art), not a patent claim.

---

## 1. What is disclosed

Three coupled mechanisms in a personal AI assistant with long-term memory (EVA):

M1 — Memory-Grounded Media Resolution.
Resolving a vague, non-catalog description of a track into a concrete track using the assistant's episodic memory of the owner, not a catalog/metadata search.

M2 — Self-Utterance Playback Parsing.
The assistant names a track inside an ordinary conversational reply (no explicit command). A separate parsing layer reads the assistant's own utterance, extracts the play intent, and executes it through the local player. Action originates from the system's own words, not from a user command.

M3 — Proactive Conversational Curation.
The assistant initiates and conducts media selection as a conversation, not as command execution: it asks the owner about his experience, offers to play from what it learns, discusses the work while it plays, proposes candidates for open requests ("something sci-fi, interesting"), answers "what is it about", and — on the owner's agreement — plays it on the appropriate device (speakers, TV). The unit of interaction is an evening spent together, not a query resolved.

All three are layered on top of the memory architecture already disclosed in this repository (episodic memory, morphological entity linking, day-anchored K-indexes, night consolidation).

---

## 2. Problem

Catalog assistants (Alexa, Alice, Spotify voice, etc.) resolve requests against a global catalog: artist, title, album, genre, metadata. They fail when the owner does not know the name — which, for most people, is the normal case. The owner remembers a fragment of the video, a life episode, a mood, "the one we listened to that evening", "the one you put on when I said the back hurts".

The missing index is not the catalog. It is the owner's own life.

## 3. Mechanism M1 — resolution through episodic memory

Input: a description containing no catalog-usable identifiers.
Examples (real, dated — see §6):
- "the white girl with the butterfly" → Die Antwoord — Enter The Ninja
- "Pet Shop Boys, the one where the three of them ride in a taxi" → Always on My Mind
- "put on something for half an hour, your taste" → a playlist assembled to fit the interval, selected against the owner's taste model (often including works he has never heard)

Resolution path:
1. The query is not sent to a catalog search.
2. Episodic memory is queried for prior conversations involving music, using entity linking (morphology-aware, inflected languages) and day-anchored indexes for temporal cues ("that evening", "yesterday").
3. Candidate episodes are returned with what the owner himself said about them (his phrasing, his reactions, the fragments he described).
4. The candidate is grounded: the answer must be supported by a stored episode, not invented. If nothing is found, the system says so.
5. The resolved track is played, and the resolution itself is recorded as a new episode.

Consequence: the owner never has to learn or remember names. The assistant is the index.

Two distinct selection modes, chosen by how the request is phrased:

(a) From the owner's own known material — "put on something of mine", "put on my favourite". Here the assistant selects from what they have actually listened to together and what the owner reacted well to. "My favourite" is not a field the owner ever set: the assistant infers it from repeated praise and repeated play of the same track over a period, and still returns it correctly weeks later, when the owner asks without naming it.

(b) Discovery by taste model — "your taste", "something for half an hour". Here selection is not a replay of the owner's listening history. The assistant builds a model of the owner's taste from behavioural and conversational signals accumulated over time — what he did not skip, what he repeated, what he skipped early, what he praised in words, what he was doing while it played — and selects against that model. Consequently it frequently plays works the owner has never heard and did not know existed, which nevertheless match his criteria. The system acts as a mirror of the owner's taste rather than as a log of his plays.

Time-bounded sessions: a request may bound the session by duration rather than by content ("half an hour while I eat"), and the assistant assembles a playlist to fit the interval. The end of the music marks the end of the break.

## 4. Mechanism M2 — the system hears itself

Standard agent pattern: user command → parser → tool call.
Disclosed pattern: the assistant, in a normal conversational reply, names a track (e.g. "Yndling — Time Time Time" on its own line). A parsing layer over the assistant's own output detects the play intent and triggers the local player. No explicit user command, no visible command syntax, no button.

Effect for the owner: the conversation and the action are one. He says "put something on"; the reply and the music arrive together.

This is not a claim over LLM output parsing or function calling in general (well-known art). The claim is narrow: the assistant's own conversational utterance, produced without command syntax, is treated as an actionable intent source, and the track named in it is one grounded in the owner's episodic memory (M1).

## 4a. Mechanism M3 — the assistant conducts, not executes

Catalog assistants wait for a command and return a result. The disclosed behaviour is a conversation that produces the media choice as a by-product, and continues around it.

Real pattern (music):
1. The assistant asks the owner about his life — e.g. whether he had been to a concert.
2. The owner mentions an artist he saw (with his daughters).
3. The assistant recognises the artist, talks about the music, and itself offers: "shall I put on their best?"
4. It plays, and the evening continues as a conversation about the artist's work and life, while the music runs.
5. Everything said that evening is stored as episodes and becomes resolvable later ("the one we listened to when we talked about the concert").

Real pattern (film / TV):
1. Owner: "I'd like some sci-fi, something interesting."
2. The assistant proposes a specific candidate — chosen against what it remembers of the owner's taste and what he has already seen.
3. Owner: "what is it about?"
4. The assistant answers, and predicts why it should suit him ("you'll like this one — it's about a takeover of the universe").
5. On agreement, it starts playback on the correct device (TV), not on the machine it is running on.

Claimed elements: the assistant asks first; it volunteers the offer; it discusses the work while it plays; it proposes candidates and defends them against remembered taste and viewing history; it routes playback to the device appropriate for the medium; and every step of that evening becomes memory, feeding future resolutions (M1).

The distinction from existing art is the direction of initiative. In catalog assistants and recommenders, the human queries and the system answers. Here the system opens the subject, and the choice emerges from a two-sided conversation grounded in what it knows about this specific person.

## 5. Difference from existing art

- Catalog voice assistants (Alexa, Alice, Siri, Spotify voice): resolve on catalog metadata; no long-term episodic memory of the owner; cannot resolve "the one with the white girl and the butterfly" unless such metadata happens to exist publicly.
- Recommender systems (Spotify/YouTube): infer taste statistically from behaviour aggregated across millions of users; they do not resolve a personal verbal description into a specific track through remembered conversations with this owner, do not answer "what did we listen to that evening", and do not hold a per-owner model built from what he said, praised and dismissed in dialogue.
- LLM agents with function calling: act on user commands parsed into tool calls; the assistant's own conversational reply is not itself the intent source, and the choice of object is not grounded in a persistent personal episodic store.

The novelty claimed here is the coupling: personal episodic memory as the resolution index (M1) + the system's own utterance as the trigger (M2).

## 6. Dated implementation evidence

- 2026-04-12: assistant recalls a listening session from the previous day, identifies the artist from a vague description ("Norwegian singer, atmospheric, cannot recall the song"), and plays the track.
- 2026-05-02: "Pet Shop Boys, the one where the three of them ride in a taxi" → Always on My Mind, resolved and played.
- 2026-05-07/08: "remember that white girl with the butterfly?" → Die Antwoord — Enter The Ninja, resolved from episodes, including the owner's own earlier remark about it.
- 2026-05-13: assistant plays a requested track directly from a conversational reply (self-utterance parsing), without the player UI being invoked by the user.
- Spring–summer 2026 (recurring, in daily use): assistant-initiated curation — it asks about a concert the owner attended, recognises the artist named in reply (Sigur Rós), offers to play their best, and the evening proceeds as a conversation about the artist's work while the music runs; for film, the owner asks for "some interesting sci-fi", the assistant proposes a specific title, explains what it is about, and starts it on the TV upon agreement.
- 2026-07-10: mechanisms described publicly in an article submitted to Habr (architecture of EVA memory).

Screenshots with system timestamps are retained by the author.

## 7. Claims (defensive)

1. Resolving an underspecified media request into a concrete item by querying the owner's personal episodic memory rather than a content catalog.
2. Using the owner's own prior utterances about an item (his description, his reaction) as the resolution key.
3. Morphology-aware entity resolution applied to media resolution in inflected languages.
4. Temporal grounding of media queries via day-anchored indexes ("that evening", "yesterday", "last week").
5. Grounded refusal: absent supporting episodes, the system states it does not know rather than inventing a match.
6. Building a model of the owner's taste from stored behavioural and conversational signals (skips, repeats, praise, context of play) and selecting against that model for open-ended requests — including works the owner has never heard, rather than replaying his listening history.
6a. Inferring "the owner's favourite" from repeated praise and repeated play over time, without the owner ever declaring it, and resolving a later unnamed request ("put on my favourite") to that track.
6b. Switching selection mode by the phrasing of the request: from the owner's own known material ("something of mine", "my favourite") versus discovery against the taste model ("your taste").
7. Duration-bounded playlist assembly, where the session length is the constraint ("half an hour while I eat").
8. Treating the assistant's own conversational utterance as an actionable play intent, parsed by a layer above the model.
9. Recording each resolution as a new episode, so that resolutions compound over time.
10. Assistant-initiated media curation: the system opens the subject, offers to play, and discusses the work while it plays, rather than awaiting a command.
11. Conversational candidate proposal for open requests, where the assistant proposes a specific work, answers "what is it about", justifies the fit against remembered taste and consumption history, and plays it on agreement.
12. Application of all of the above across media types (music, film, video) with routing of playback to the device appropriate to the medium (speakers, TV).
13. The combination of any of the above with the memory layers disclosed in this repository (Soul File, versioned facts, episodic memory, entity linker, K-indexes, night consolidation).

## 8. Status

Working implementation in daily personal use since spring 2026. This document is a defensive publication to establish prior art with a public date. It is not a patent application and does not restrict others; it prevents anyone from patenting the disclosed mechanisms against the author.
