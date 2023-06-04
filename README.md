# Ideas for Reddit on ATProto (v2)

This repo contains lexicon schemas and some documentation on how a Reddit-like social network could be built on top of ATProto.

The schema examples use the `red.solver.board` namespace, that's temporary.

All schemas are in the `lexicons/` directory of this repo.

## Open Issues

- Members can't delete votes, submissions, comments or leave a board if the board decides to not cooperate

### Board

Subreddits are called boards. Every board needs a DID and ATProto repo.

All records relevant to a board (profile, member, submission, vote, comment) are stored in the board DID's repository, most of them are signed by the DID of the user doing the operation too (member, submission, vote, comment).

Every board is running a service to provide all relevant APIs for users.

### Board Profile

Boards can have a short `displayName`, a short `summary` and a long `description` with detailed info and rules for that specific board. Boards can also have an `avatar` and a `banner` image, just like profiles.

> [TODO: update section with better solution]
> Viewing submissions in a board is handled by feed generators. The board record has a `feeds` array which references one or multiple `app.bsky.feed.generator` records. The first one is the default one shown when viewing a new board for the first time. These feed generators can provide well-known views like "Hot", "New", "Rising", "Top (with time range)" and "Controversial", but also completely new and interesting algorithms.

### Member

A member record indicates that a user is a member of a board. To leave a board, the record is deleted. Member records can contain an optional `role` field to set permissions for a user. `moderator` is needed to hide or delete submissions and ban members, the `admin` permission is required to add new moderators and update the board itself. All moderation actions should be transparent and records too, this is not specced yet.

Member records live in the board's repo, but MUST be signed by the member's DID too. They MUST include both the member DID and board DID to prevent other boards from re-using signed records to prove that someone is a member of a different board. 

### Submission

Users can submit submissions to boards if they are a member. Submissions do NOT contain text, images or other media directly. Instead, they reference another `subject`, which could be a Bluesky post or a record from another new lexicon, for example video. This architecture makes it possible to submit the same record to multiple boards without duplicating the content itself. It's also pretty powerful, because it could make anything from videos, podcasts to 3D models or books possible to be referenced and submitted without the board lexicon needing to be extended.

In addition to the subject, submissions specify a `createdAt` timestamp.

Submission records live in the board repo, but MUST be signed by the submitters's DID too. The record MUST contain the board DID and author DID.

### Upvotes and downvotes

Upvotes and downvotes work exactly like app.bsky likes, they just have a different name and reference submissions instead of posts or feed generators. Only members of a board can upvote and downvote submissions in it, invalid votes are just ignored by aggregators.

### Comment

A comment is a text-only comment on a submission or another comment. The `text` field supports Markdown formatting and has a max size of `10000` bytes or graphemes. `root` always references the root submission, `parent` is optional and references the parent comment.

## Missing Reddit features

- Moderation (should be compatible with bsky)
- Post and user flairs
- ...