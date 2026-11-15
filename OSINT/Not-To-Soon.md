# Not to Soon


- **Category:** OSINT

---

## Challenge Description

Three weeks ago, a quiet account went live. No bio, no followers, no face. Just a handful of posts and one message repeated again and again: "not so soon."

Then, a few days later, the tone shifted. The captions started to read like a countdown. Whoever is behind it wants something to be found, but only by people who look closely.

The challenge provided an Instagram account to investigate:

`@not_so_soon_`

The instructions were:

- Read everything.
- Not every message is meant to be seen at first glance.

## Investigation

I started by visiting the Instagram account mentioned in the challenge.

The account contained several posts, so I began inspecting them individually rather than relying only on the profile information.

One post, dated **October 23**, caught my attention.

It contained an AI-generated image with a dark background. Looking closely at the image, I noticed text embedded in it:

```text
but_f0und_1t}
```

This looked like the second half of the flag.

## Identifying the Flag

The challenge title was **Not to Soon**, and the Instagram account name was `@not_so_soon_`.

I suspected that the title and account name represented the first part of the flag.

Combining the two fragments gave me an initial guess:

```text
InIt{not_so_soon_but_f0und_1t}
```

However, submitting this version returned an incorrect flag.

I then noticed that the challenge's image already used numbers in place of letters. For example:

- `f0und` uses `0` instead of `o`.
- `1t` uses `1` instead of `i`.

Following the same pattern, I replaced the letter `o` in the first part of the flag with the number `0`.

The corrected flag was accepted.

##  Final Flag

```text
InIt{n0t_s0_s00n_but_f0und_1t}
```

##  Key Takeaways

- Inspect every post on an account rather than focusing only on its profile.
- Pay attention to captions and image text. They may contain separate clues.
- Look closely at unusual character substitutions, especially letters replaced by numbers.
- When a flag looks correct but is rejected, reconsider its exact spelling and formatting rather than immediately assuming the clue is wrong.
