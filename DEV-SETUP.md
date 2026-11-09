# Building a development nest for BigFix content

This document describes setting up an environment for developing BigFix Content.
While these notes assume use of MS VSCode, many of the notes herein should be portable to other IDEs.

# Overview

* Setup Git Repo/Submodule Hierarchy
* Setup BigFix MCP
* Setup Agent Skills submodules, AGENTS.md, CLAUDE.md
* Setup default git things
* Setup default VSCode configs
* Setup Python
* Setup pre-commit

# Configure Git Submodule Hierarchy

It is very useful to have a single instance of VSCode configuration, .gitattributes, AI skills, and AI prompts, that are up-to-date and usable in each repository.  This is especially useful when managing multiple repositories of similar content (like BigFix Sites).

However much of the AI tooling is based on using specific filenames and paths relative to a git repository, such as top-level '.agents' and '.vscode' directories.

The solution to this is generally to use 'git submodules'.  In this way, there is a top-level repository with bare scaffolds of AI tooling, and each real content repository is treated as a submodule of the parent repo.  Each submodule has its own git commit history, its own branches, its own pull-request flow, and can be updated independently of the parent repo.
In fact, the parent repo does not even require an upstream, if we do not need to track changes to it.

For more on 'git submodule' usage, see [TODO]

Here, assuming we have a directory 'repository_sites' which will contain three distinct git repositories beneath it, i.e.

```none
repository_sites/
    site_BigFix_Management
    site_Hardware_Management
    site_Rapid_Response
```

We can implement this by the following:

* First, initialize 'repository_sites' as a git repo:
`c:\repository_sites> git init`
* Next, create submodules cloned from each parent repository:

```
c:\repository_sites> git submodule https://github.com/myorg/site_BigFix_Management.git

c:\repository_sites> git submodule https://github.com/myorg/site_Hardware_Management.git

c:\repository_sites> git submodule https://github.com/myorg/site_Rapid_Response.git
```

## Sharing and cloning the hierarchy

If the top-level 'repository_sites' does have a git upstream repository, and you wish to share the full hierarchy amongst teammates, the top-level and all submodules can be cloned via
`git clone --recurse-submodules <parent-repo-url>`

If the submodules list changes over time, each person can update their repos list with the current via
`git submodule update --init --recursive`

# Setup BigFix MCP

Recommended: first install the 'uv' python package manager, see <https://docs.astral.sh/uv/getting-started/installation/>

Then install the open-source 'bigfix-root-mcp' from jgstew:
<https://github.com/jgstew/bigfix-root-mcp>

See the README.md for full details and setup instructions

```none
# Setup:
uv tool install bigfix-root-mcp
# This installs it to an executable in the PATH, for instance
# C:\Users\user.name\.local\bin\bigfix-root-mcp.exe

# the tool can be invoked via
bigfix-root-mcp

# Update
uv tool upgrade bigfix-root-mcp
```

Register the MCP into the workspace by editing .mcp.json (or ask your AI agent how to register it)

`mcp.json`:

```json
{
    "mcpServers": {
        "bigfix-root": {
            "command": "bigfix-root-mcp",
            "args": [],
            "env": {
                "BES_ROOT_SERVER": "https://<ROOT_SERVER_HOST>:52311",
                "BES_USER_NAME": "<BES_USER_NAME>",
                "BES_PASSWORD": "<BES_PASSWORD>"
            }
        }
    }
}
```

or, the default for a non-persistent MCP usage:

```json
{
  "mcpServers": {
    "bigfix-root": {
      "command": "uvx",
      "args": ["bigfix-root-mcp"],
      "env": {
        "BES_ROOT_SERVER": "https://<ROOT_SERVER_HOST>:52311",
        "BES_USER_NAME": "<BES_USER_NAME>",
        "BES_PASSWORD": "<BES_PASSWORD>"
      }
    }
  }
}
```

If you already have besapi set up, remove the environment variables from the 'env' stanza, and your existing .besapi.conf will be used instead.

When the mcp is configured and working, you can ask about it in chat such as

```
"ask bigfix-root-mcp whoami"

Connected successfully.

User: <my username>
Server: <my root server>
Master operator: Yes — full deployment visibility
Write tools: Disabled
QnA: Enabled
MCP version: 1.2.0
```

# Setup Agent Skills submodule, AGENTS.md, CLAUDE.md

Create the expected directory for AI Agent Skills in the top-level repository.  Create a submodule linking an Agent Skill into the skills directory.

```none
C:\git\repository_sites> mkdir .agents\skills
C:\git\repository_sites> git submodule add https://github.com/Jwalker107/bigfix-skill.git .\.agents\skills\bigfix-skill
```

Create a minimal AGENTS.md in the top-level of the repo.
Create a CLAUDE.md that references AGENTS.md so Claude gets on the same page as everyone else.

# Setup git defaults

The '.gitattributes' file should be created in both the top-level git repo, and in each of the submodules.  Following is suggested content that configures syntax highlighters or auto-formatting tools to respect the expected file types.  The entries for .ojo, .beswrpt, and .bes are important for BigFix content.

```
# https://github.com/github/linguist/blob/master/lib/linguist/languages.yml
# https://github.com/github/linguist/blob/master/docs/overrides.md
*.bes -text eol=CRLF linguist-language=XML linguist-detectable
*.ojo linguist-language=XML linguist-detectable
*.beswrpt linguist-language=HTML linguist-detectable
*.html linguist-language=HTML
*.xml linguist-language=XML linguist-detectable
# https://stackoverflow.com/a/68932262
*.reg working-tree-encoding=UTF-16 eol=CRLF
```

# Setup default VSCode configs

Setting up some default configuration for MS VSCode is helpful for ensuring a common environment for everyone who shares the repo.
Things that can be applied include listing recommended Extensions for this repo, configuration of Python virtual environments for working with scripts in the repo.

Check the '.vscode' directory in this repo for some examples.  In particular we like using .vscode/extensions.json to recommend the 'jgstew.bigfix-relevance-developer' extension, 'tasks.json' to automatically create a Python virtual environment for the top-level repo, and a 'settings.json' to *use* that new virtual environment.

# Setup Python

Speaking of setting up Python, we often set up Python for a couple of reasons.  It is useful in setting up Github Actions, useful in validating content using pre-commit hooks, and useful in many of the AI tooling scenarios.
Setting up Python is now largely included in the 'vscode/tasks.json', if you're not using VSCode then check there for advice on setting up Python manually or in your own IDE.

# Setup pre-commit

<https://pre-commit.com/>

Run one of these depending on your system type.  This only needs to be run once.

```bash
# Using pip (Python)
pip install pre-commit

# Using Homebrew (macOS)
brew install pre-commit

# Using Conda
conda install -c conda-forge pre-commit
```

In the parent repo and in each child repo, create a `.pre-commit-config.yaml` to track which hooks should be run every time we attempt to 'commit' into the repo:

Check the `.pre-commit-config.yaml` file for examples.  In particular we recommend including the hooks from `https://github.com/jgstew/pre-commit-bigfix` to automatically validate BigFix content.
The `bes-siterepo-validate-layout` hook should be enabled on RepositorySites (that are meant to be direct BigFix content to gather) but should not be enabled on the parent repo or other site types.

In *each* repository (the parent, and every submodule), install the pre-commit hooks via
`pre-commit install`
