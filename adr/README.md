# Architecture decision records

A decision record captures a choice that outlives the pull request that made it:
why one option was taken, what was given up, and what would have to change for the
decision to be revisited.

Write one when the answer to "why is it like this" would otherwise live only in
someone's head or in a review thread - a module boundary, a client contract, a
versioning rule, a dependency the whole space now carries. Do not write one for a
bug fix or a refactor with an obvious motivation.

## How to add one

1. Copy [`template.md`](template.md) to `NNNN-short-title.md`, where `NNNN` is the
   next free number.
2. Fill it in. Keep it short - a page is usually enough, and a long record is a
   sign the decision is not settled yet.
3. Open a pull request. The review is about the decision, not the prose.

A record is immutable once it is accepted. Changed your mind? Write a new record
that supersedes it and mark the old one `Superseded by NNNN`. The history of
rejected reasoning is the part that saves time later.

## Records

| Number | Title | Status |
| :--- | :--- | :--- |
| - | No records yet | - |
