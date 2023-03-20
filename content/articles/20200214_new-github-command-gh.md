---
title : New Github Official CLI `gh`
tags : ["gh"]
date : 2020-02-14
published : true
---

[github.com/cli/cli/cmd/gh](https://github.com/cli/cli/cmd/gh) is new official command for GitHub.

<!--more-->

# How to install `gh` command

```sh
go get github.com/cli/cli/cmd/gh
```

# How to use `gh issue`?

`gh issue` have subcommands `create`, `list`, `status`, `view`.

If you want to create new issue, that's below.

```sh
cd your-project/your-repo
gh issue create
#  or
gh issue create -R your-project/your-repo
```

Then you ask to authorize `gh` command to access your github account.

After you have authorize, input title and body for issue and choose to (Check in browser|Submit|Cancel).


```sh
Creating issue in your-project/your-repo
? Title this is new issue
? Body <Recieved>  <- input by $EDITOR(in my case, vim)
? What's next? Submit
```

Done to create new issue.


# How to use `gh pr`?

`gh pr` have subcommands `checkout`, `create`, `list`, `status`, `view`.

If you want to create new pull-request, that's below.

```sh
cd your-project/your-repo
gh pr create
#  or
gh pr create -R your-project/your-repo


Creating pull request for <from-branch> into <to-branch> in your-project/your-repo
? Title this is test `gh`
? Body <Recieved>  <- input by $EDITOR(in my case, vim)
? What's next? Submit
```

Done to create new pull-request.

