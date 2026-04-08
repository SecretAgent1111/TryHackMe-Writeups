# Hidden Deep Into My Heart — TryHackMe Writeup

Just finished the **Hidden Deep Into My Heart** room from Love at First Breach 2026. This one
was mostly about directory bruteforcing and finding hidden paths, so it felt like a classic web
recon challenge where the main job is just not giving up too early.

---

## Room Details

| Field | Info |
|-------|------|
| Name | Hidden Deep Into My Heart |
| Difficulty | Entry-Level |
| Category | Web Security / Directory Enumeration |
| Platform | TryHackMe |

---

## Initial Setup

I started the machine normally and grabbed the target IP first. After that, I did a quick ping
check just to make sure the box was actually up and reachable.

```bash
ping <target-ip>
```

Everything looked good, so I moved on to the website.

---

## First Look

I opened the IP in my browser to see what the landing page looked like. It was a pretty simple
page, nothing flashy, and there wasn't much exposed on the surface.

I also checked the page source and looked at `robots.txt` out of habit, but nothing useful
stood out right away.

---

## Directory Bruteforcing

This is where the room really starts. The title basically gives away what you're supposed to
do — find what's hidden deep in the directories.

I started with a common wordlist first:

```bash
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirb/common.txt
```

That gave me a few results, but nothing too exciting yet. So I switched to a bigger wordlist
and added a few extensions just in case there were files hiding behind them.

```bash
gobuster dir -u http://<target-ip> \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,html,txt
```

That scan took a bit longer, but it started showing me more useful directories.

---

## Going Deeper

Once I found a directory that wasn't linked anywhere on the main page, I started checking
inside it manually.

```bash
curl http://<target-ip>/hidden-directory
```

Some of those directories had more content inside them, so I kept scanning deeper instead
of stopping after the first hit.

```bash
gobuster dir -u http://<target-ip>/found-directory \
  -w /usr/share/wordlists/dirb/common.txt
```

That's really the whole idea behind this room — it's less about fancy exploitation and more
about being persistent with enumeration.

---

## Finding The Path

After going through multiple layers of directories, I finally found a hidden path that wasn't
obvious from the main site at all.

What helped most was:

1. Not stopping after the first scan.
2. Checking every discovered directory for more paths.
3. Using different wordlists when needed.
4. Being patient and methodical.

This room was basically a reminder that a lot of web challenges are solved by careful digging,
not by rushing.

---

## Getting The Flag

Once I reached the final hidden directory, I opened it in the browser and found the flag file there.

```bash
curl http://<target-ip>/very/deep/hidden/path/flag.txt
```

The flag format was `THM{...}`.

After submitting it, the room was complete.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| GoBuster | Directory enumeration |
| cURL | Quick checks on discovered paths |
| Browser | Manually inspecting pages |
| DirBuster wordlists | Deeper brute forcing |

---

## What I Learned

- Directory bruteforcing is a core web pentesting skill.
- Hidden paths often sit several levels deep.
- Different wordlists can reveal different things.
- Patience matters more than speed.
- Not everything useful is visible in the source or `robots.txt`.

---

## Reflection

Overall, this was a solid beginner web room. It wasn't difficult, but it was good practice for
building the right enumeration mindset.

In real engagements, the same approach can uncover admin panels, backup files, config pages, and
other hidden endpoints that are easy to miss if you only look at the homepage.
