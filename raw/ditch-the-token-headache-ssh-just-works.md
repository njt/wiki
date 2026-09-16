---
url: https://www.shankuehn.io/post/ditch-the-token-headache-ssh-just-works
date_fetched: 2026-09-16
---

# Ditch the Token Headache...SSH Just Works!

If you've pushed to GitHub and gotten smacked with remote: Support for password authentication was removed, you're not alone, and you're not doing anything wrong. GitHub __officially retired password auth over HTTPS back in August 2021__ in favor of token- or SSH-based authentication. It's not a bug, it's not your firewall, it's just the old way not working anymore.

I'd been getting by on Personal Access Tokens, but fine-grained PATs come with their own overhead: picking scopes per repo, watching expiration dates, regenerating them when they lapse mid-project. For someone spinning up a lot of repos, that's unfortunately a lot of bookkeeping for something that should just work. I wanted a more straightforward proposition: authenticate once, and be done with the whole thing!

The good news: fixing it takes about five minutes, and if you set it up right, you won't have to think about it again.

## Two paths forward

So know this outright, you've got two options to choose here (I'm heavily advocating for one over the other): a Personal Access Token (PAT) over HTTPS, or SSH keys. Both work. I'm going to make the case for SSH, because if you're tired of managing fine-grained token scopes and expiration dates across a lot of repos (like I was), you want something you configure once and forget. I'm perpetually the lazy sysadmin.

PATs are definitely great for CI pipelines or scripts where you need scoped, expiring access. But for day-to-day work, they mean picking permissions per token and re-authenticating whenever one expires. SSH just sits there and works, which means it's the de facto heavyweight champ!

## Setting it up

First, check if you already have a key kicking around:

`ls -al ~/.ssh`Looking for id_ed25519 and id_ed25519.pub (or the older id_rsa pair). If you've got one you're already using elsewhere, you can skip straight to adding it to GitHub. If not, generate one:

`ssh-keygen -t ed25519 -C "[email protected]"`Use the email tied to your GitHub account. Hit Enter to accept the default save location. A passphrase is optional but recommended if this is a machine other people can touch.

Next, get the agent running and load your key into it:

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
On a Mac, add --apple-use-keychain to that ssh-add command so it survives a reboot without asking you to reload the key.

Now grab your public key:

`cat ~/.ssh/id_ed25519.pub`Or, if you're on macOS, skip the copy-paste dance entirely:

`pbcopy < ~/.ssh/id_ed25519.pub`Head to GitHub, go to **Settings → SSH and GPG keys → New SSH key**, give it a name that'll actually mean something to you in six months (your laptop's hostname is fine), and paste it in.

Test the connection:

`ssh -T `__[email protected]__You'll get a prompt about an unrecognized host the first time. Type yes. You should land on a message confirming who you're authenticated as.

## Why this covers every repo, not just one

This is the part that trips people up: SSH auth is keyed to the *host* (github.com), not to individual repositories. Once your key is registered with your GitHub account, it authenticates you for every repo you own, contribute to, or clone, forever, with no per-repo setup.

The only thing you need to do per repo is make sure it's actually using an SSH remote instead of an HTTPS one. For anything you've already cloned over HTTPS:

```
git remote set-url origin [email protected]:username/repo.git
git remote -v
```
That second command just confirms the switch took. For anything new, clone with the SSH URL from the start:

`git clone [email protected]:username/repo.git`On the repo's GitHub page, the green **Code** button lets you toggle between HTTPS and SSH so you're copying the right URL without having to remember the syntax.

## The takeaway

Password auth for git isn't coming back, and honestly, SSH is the better long-term move anyway. Spend the five minutes once, and you're done thinking about git authentication for every repo you'll ever create.
