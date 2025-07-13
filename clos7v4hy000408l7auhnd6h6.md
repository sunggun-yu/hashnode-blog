---
title: "How to use multiple GitHub accounts when only HTTPS is allowed"
seoTitle: "how to use multiple GitHub account in HTTPS"
datePublished: Fri Nov 10 2023 06:08:04 GMT+0000 (Coordinated Universal Time)
cuid: clos7v4hy000408l7auhnd6h6
slug: how-to-use-multiple-github-accounts-when-only-https-is-allowed
tags: github, git, git-https, git-multiple-account

---

*It was relatively simple when I could use GIT+SSH...*

I have separate GitHub user accounts for my personal and company use, and I could configure Git to use specific SSH keys for different URL patterns using `includeIf` as below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699553876922/aa10e430-6c39-4a01-b114-b6997ac50ee2.png align="center")

`~/.gitconfig` file:

```ini
[user]
	name = Sunggun Yu
	email = sunggun@my-company.com
[core]
	excludesfile = ~/.gitignore_global
[alias]
	co = checkout
	br = branch
	ci = commit
	st = status
## my work github org path in git+ssh
## You can skip this sction to make your work account the default for all GitHub access in your work machine
[includeIf "hasconfig:remote.*.url:git@github.com:my-work-org/**"]
	path = ~/.config/gitconfig/work/.gitconfig
## my personal github org/user path in git+ssh
[includeIf "hasconfig:remote.*.url:git@github.com:my-account/**"]
	path = ~/.config/gitconfig/personal/.gitconfig
```

`~/.config/gitconfig/work/.gitconfig` file:

```ini
[user]
	name = Sunggun Yu
	email = sunggun@my-company.com
[core]
  sshCommand = "ssh -i ~/.ssh/my-work-private-key"
```

`~/.config/gitconfig/personal/.gitconfig` file:

```ini
[user]
	name = Sunggun Yu
	email = sunggun@my-personal.com
[core]
  sshCommand = "ssh -i ~/.ssh/my-personal-private-key"
```

However, the company has configured a company-wide security proxy on our computers, which blocks TCP port 22. This means I can no longer use Git with SSH and must switch to using Git with HTTPS.

You can obviously clone or set remote URL with HTTPS using a Username and Password. If you attempt to clone via HTTPS for the first time, you will be prompted to enter a username and password.

> The password to enter is PAT(Personal Access Token).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699580556624/765139a6-2e6b-423e-a542-897b1b7b9b36.png align="center")

You won't be asked for these credentials again when cloning another repository accessible with the same username.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699580417791/bc3894c5-b8a5-4a24-a653-309375238f86.png align="center")

This is because git uses the `osxkeychain` [gitcredential](https://git-scm.com/docs/gitcredentials) helper by default when you're using HTTPS on MacOS

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699582293688/10baaa6a-5b22-413a-8659-b100e7436f84.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699581962000/6acb04d8-cd70-4100-819a-4d1dd1ec4688.png align="center")

The issue is it will fail if you try to clone another repository that is accessible only by a different account

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699579311773/04da92ad-1277-4641-a242-b33a0fd30a5d.png align="center")

## Solution

So, you should let git know which credential in Keychain needs to be used for each Git URL

```bash
# hey Git, use `my-personal-github-id` as username
# when git url is starting with https://github.com/my-account
git config --global \
  credential.https://github.com/my-account.username \
  my-personal-github-id

# hey Git, use `my-work-github-id` as username
# when git url is starting with https://github.com/my-work-org
git config --global \
  credential.https://github.com/my-work-org.username \
  my-work-github-id
```

As a result, the `credential` configuration is added in `~/.gitconfig` as below, given that we use the `--global` option.

```ini
[user]
	name = Sunggun Yu
	email = sunggun@mycompany.com
[core]
	excludesfile = ~/.gitignore_global
[alias]
	co = checkout
	br = branch
	ci = commit
	st = status

[credential "https://github.com/my-work-org"]
	username = my-work-github-id
[credential "https://github.com/my-account"]
	username = my-personal-github-id
```

As shown below, Git will utilize the associated username when the URL pattern is matched.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699589135859/908b9e3c-703c-4645-b6ef-9852689e6e26.png align="center")

Now, you will see that another [GitHub.com](http://GitHub.com) internet account has been added to Keychain

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699589714561/85da51ab-cba0-4d5e-9101-626a371dad46.png align="center")

We can leverage `includeIf` with a URL pattern similar to what we did for `git+ssh`. but please be aware of the format difference between `git+ssh` and `https`

> `git@github.com:my-account` vs.`https://github.com/my-account`

`~/.gitconfig` file:

```ini
[user]
	name = Sunggun Yu
	email = sunggun@my-company.com
[core]
	excludesfile = ~/.gitignore_global
[alias]
	co = checkout
	br = branch
	ci = commit
	st = status
## my work github org path in git+ssh
## You can skip this sction to make your work account the default for all GitHub access in your work machine
[includeIf "hasconfig:remote.*.url:https://github.com/my-work-org/**"]
	path = ~/.config/gitconfig/work/.gitconfig
## my personal github org/user path in git+ssh
[includeIf "hasconfig:remote.*.url:git@github.com:my-user/**"]
	path = ~/.config/gitconfig/personal/.gitconfig
```

`~/.config/gitconfig/work/.gitconfig` file:

```ini
[user]
	name = Sunggun Yu
	email = sunggun@my-company.com
[credential "https://github.com"]
	username = my-work-github-id
```

`~/.config/gitconfig/personal/.gitconfig` file:

```ini
[user]
	name = Sunggun Yu
	email = sunggun@my-personal.com
[credential "https://github.com"]
	username = my-personal-github-id
```

You may noticed individual config doesn't include path in `credential` host.

```ini
[credential "https://github.com"]
```

Additionally, I use [`git-credential-manager`](https://github.com/git-ecosystem/git-credential-manager) for more alternative authentification methods.

```bash
# Install git-credential-manager
brew install --cask git-credential-manager
```

```bash
# Configure global ~/.gitconfig, to use git-credential-manager
git credential-manager configure
```

## Set a default account

You may be wondering how I can set my personal GitHub account as the default while having my work account as an option in my personal environment or on my personal devices.

It is possible but it might be tricky. the most important step is to log in to your personal account or the one you intend to set as the default account first 😛. You may also consider cleaning up existing `github.com` accounts in `Keychain Access`.

### How to remove keychains

1. open `Keychain Access` &gt; select `Open Keychain Access` if select dialogue pops up
    
2. search `github.com`
    
3. remove `github.com` internet accounts
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699594754533/c23e8300-cc9b-43cd-b765-75c0c421ee79.png align="center")

### Configure gitconfig

`~/.gitconfig` file:

```ini
[user]
	name = Sunggun Yu
	email = sunggun@my-personal.com
[core]
	excludesfile = ~/.gitignore_global
[alias]
	co = checkout
	br = branch
	ci = commit
	st = status

## my work github org path in git+https
## You can skip this sction to make your work account the default for all GitHub access in your work machine
[includeIf "hasconfig:remote.*.url:git@github.com:my-work-org/**"]
	path = ~/.config/gitconfig/work/.gitconfig
## use git-credential-manager as credential helper
[credential]
	helper = /usr/local/share/gcm-core/git-credential-manager
```

`~/.config/gitconfig/work/.gitconfig` file:

```ini
[user]
	name = Sunggun Yu
	email = sunggun@my-company.com
[credential "https://github.com"]
	username = my-work-github-id
```

### Login to personal first

Login to a personal account by cloning some GitHub repository

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699595059440/9ef49a67-9953-443c-9eb2-1cdca960fbc5.png align="center")

### Then, login to the work account

Login to a work account by cloning the GitHub repository in the work organization

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699596095804/5bc430e8-7abc-4fcc-93bf-84ed007ecb86.png align="center")

## References

* [https://git-scm.com/docs/gitcredentials](https://git-scm.com/docs/gitcredentials)
    
* [https://git-scm.com/docs/gitcredentials#\_custom\_helpers](https://git-scm.com/docs/gitcredentials#_custom_helpers)
    
* [https://github.com/git-ecosystem/git-credential-manager](https://github.com/git-ecosystem/git-credential-manager)
    
* [https://github.com/git-ecosystem/git-credential-manager/blob/main/docs/multiple-users.md](https://github.com/git-ecosystem/git-credential-manager/blob/main/docs/multiple-users.md)
    
* [https://github.com/git-ecosystem/git-credential-manager/blob/main/docs/credstores.md](https://github.com/git-ecosystem/git-credential-manager/blob/main/docs/credstores.md)