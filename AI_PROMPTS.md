# Generative AI Disclosure

Using AI tools for this challenge is allowed. Be honest here — an accurate
"I used AI heavily" is worth more to us than an inaccurate "I did not".

**Did you use AI tools?** (yes / no): yes

**Which ones?** (e.g. ChatGPT, Claude, Copilot, Cursor): Gemini

> If you answered **no**, write "N/A" in sections 1 and 2 and skip to
> section 3. That is a perfectly good answer.

---

## 1. Key prompts

List the main prompts you used. Paraphrasing is fine.

1. `Explain Docker Compose internal DNS and how to resolve NGINX 502 Bad Gateway errors.`
2. `How to configure a Docker Compose healthcheck to handle container startup race conditions.`
3. `Fixing 'Permission denied (exit code 126)' for a bash script in GitHub Actions.`

---

## 2. Where the AI got it wrong

Describe **one concrete case** where the AI's answer was wrong, outdated, or
sub-optimal. Cover: what it suggested, how you noticed it was wrong, and what
you did instead. 3–5 sentences.

> During Task 1, the proxy Docker image failed to build because the `apt-get install nginx` command was blocked by the pre-existing `nginx.conf` file. The AI suggested modifying the `apt-get` command with force flags to bypass the interactive prompt and overwrite the file. I realized this was a "brute force" approach and not a best practice. Instead of forcing the installation, I simply changed the order of instructions in the `Dockerfile` so that `RUN apt-get install` happens before `COPY nginx.conf`, which cleanly solved the root cause.

---

## 3. Anything in your submission you could not fully explain?

We would rather know. Write "nothing" if that is the case.

> nothing
