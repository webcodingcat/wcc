---
name: prototype
description: Build a quick throwaway prototype so the user can find out what they actually want before the idea interview or the spec. Use this whenever the user cannot answer interview questions because they have never seen the thing, wants to "just try something out", or is planning a game or UI whose rules or feel are still unclear.
argument-hint: "<what to try out>"
---

# prototype: build to learn, then throw away

What to try out: $ARGUMENTS

Many interview questions cannot be answered about something nobody has seen
yet. A prototype exists to produce those answers. It is not the start of the
project, and none of its code is carried over.

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in. A
call without text carries no language: then use the configured one, or the
language of the conversation so far, never the language of the documents or
of this skill, which are English.

## Rules

- Build in the folder `prototype/` in the project root and nowhere else.
- Aim for something the user can try within one to two hours of work. Cut
  everything that does not help answer the open questions.
- No planning documents, no tests, no reviews, no architecture. Pick whatever
  stack gets to a runnable result fastest; this choice says nothing about the
  stack of the real project.
- Ask at most one round of questions before starting, only to find out which
  open questions the prototype should answer.

## Afterwards

1. Tell the user how to start the prototype and what to try.
2. When they have tried it, ask what they learned, in the interview format
   (options with a recommendation where possible):
   - What must stay exactly like this?
   - What felt wrong?
   - What was missing?
3. Write the answers to `.project/decisions/prototype-learnings.md` in English.
4. Ask whether `prototype/` can be deleted. Never import or copy code from it
   into the project, even if it is kept.
5. Point to the next step: `/wcc:idea`, which reads
   `.project/decisions/prototype-learnings.md` if it exists, or `/wcc:change`
   if the project already has an IDEA.md.
