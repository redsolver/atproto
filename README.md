# Ideas for Reddit on ATProto

This repo contains lexicon schemas and some documentation on how a Reddit-like social network could be built on top of ATProto.

But let's start with an idea on how Reddit-style features could be built using only the `app.bsky` lexicon:

- Subreddits are just bsky actors with some extra metadata
- Views like "Hot", "New" or "Top" are just custom feeds published by the actor
- Users can join the subreddit by following the actor
- Users can be banned from a subreddit by being blocked
- Users can submit their post to one or multiple subreddits by mentioning the subreddit actor handle somewhere in their post
- Comments are just replies on the post
- Likes from members of a subreddit (following) are counted at upvotes for the subreddit-specific feeds

### Advantages

- Works with any existing Bluesky app
- No new lexicons needed

### Disadvantages

- Post and comment text size limited to 300 chars (graphemes)
- No downvotes
- No subreddit-specific discussion, all replies are global
- Every subreddit needs a new account
- Subreddit-specific posts and replies could spam bluesky feeds

## Alternative: Concept for new Reddit-only lexicon

The schema examples use the `red.solver.board` namespace, that's temporary.

All schemas are in the `lexicons/` directory of this repo.

### Board

Subreddits are called boards.

DIDs/actors can create multiple boards and boards are bound to their account. Similar to feed generators, the rkey defines the URI of the board and how it looks in URLs.
So for example `at://example.com/red.solver.board.board/cats` could be a board with the slug `cats`, created by `@example.com`. A board with the rkey `main` is the default board for a specific actor and should be displayed prominently in the UI when viewing a specific handle. This makes it possible to define boards for domains.

Boards can have a short `displayName`, a short `summary` and a long `description` with detailed info and rules for that specific board. Boards can also have an `avatar` and a `banner` image, just like profiles.

Viewing submissions in a board is handled by feed generators. The board record has a `feeds` array which references one or multiple `app.bsky.feed.generator` records. The first one is the default one shown when viewing a new board for the first time. These feed generators can provide well-known views like "Hot", "New", "Rising", "Top (with time range)" and "Controversial", but also completely new and interesting algorithms.

### Join

A join record is created by an actor to join a board as a member. Deleting it leaves again.

### Submission

Users can submit submissions to boards if they are a member. Submissions do NOT contain text, images or other media directly. Instead, they reference another `subject`, which could be a Bluesky post or a record from another new lexicon, for example video. This architecture makes it possible to submit the same record to multiple boards without duplicating the content itself. It's also pretty powerful, because it could make anything from videos, podcasts to 3D models or books possible to be referenced and submitted without the board lexicon needing to be extended.

In addition to the subject, submissions specific the URI of the `board` they are submitting to and a `createdAt` timestamp

### Upvotes and downvotes

Upvotes and downvotes work exactly like app.bsky likes, they just have a different name and reference submissions instead of posts or feed generators. Only members of a board can upvote and downvote submissions in it, invalid votes are just ignored by aggregators.

### Comment

A comment is a text-only comment on a submission or another comment. The `text` field supports Markdown formatting and has a max size of `10000` bytes or graphemes. `root` always references the root submission, `parent` is optional and references the parent comment.

## Open questions

- Boards are bound to the DID which created it and transferring them is not possible. Maybe there's a better solution?

## Missing Reddit features

- Moderation (should be compatible with bsky)
- Post and user flairs
- ...