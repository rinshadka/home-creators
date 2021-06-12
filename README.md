# Netspective Studios Typical Polyglot Creator's Home Setup

Shahid's typical Engineering Sandbox for polyglot software development or any other "creator tasks" that are performed on Linux-like operating systems.

If you're using Windows 10 with WSL2, create a "disposable" Debian WSL2 instance using Windows Store. This project treats the WSL2 instance as "disposable" meaning it's for development only and can easily be destroyed and recreated whenever necessary. The cost for creation and destruction for a Engineering Sandbox should be so low that it should be treated almost as a container rather than a VM.

If you're using a Debian-based distro you should be able to run this repo in any Debian user account. It will probably work with any Linux-like OS but has only been tested on Debian-based distros (e.g. Debian and Ubuntu).

## One-time setup

### One-time setup of typical utilities

Bootstrap a Debian environment with required utilities (if you're using another distro, use your own package commands):

```bash
# assume bash as the default shell, requires sudo (admin) privileges
sudo apt-get -qq update
sudo apt-get -y -qq install curl git jq pass unzip bzip2 tree make
curl -sSL https://git.io/git-extras-setup | sudo bash /dev/stdin
```

Install `just` and `asdf` (these should work in any distro, not just Debian):

```bash
# assume bash as the default shell, normal user privileges should work
curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to $HOME/bin
ASDF_VERSION=`curl -s https://api.github.com/repos/asdf-vm/asdf/tags | jq '.[0].name' -r` \
    bash -c 'git -c advice.detachedHead=false clone --quiet https://github.com/asdf-vm/asdf.git ~/.asdf --branch ${ASDF_VERSION}'
. $HOME/.asdf/asdf.sh
for pkg in direnv deno; do asdf plugin-add $pkg; asdf install $pkg latest; asdf global $pkg latest; done
```

* We use `$HOME/bin` for binaries whenever possible instead globally installing them using `sudo`. Be sure to add `$HOME/bin` to your path.
* We use `git` and `git-extras` because we're a GitOps shop.
* We use [just](https://github.com/casey/just) command runner to execute tasks (available in `$HOME/bin`).
* We use [asdf](https://asdf-vm.com/) as our package manager to install all languages and utilities possible so that they can be easily installed and, more importantly, support multiple versions simultaneously.
* We use [pass](https://www.passwordstore.org/) the standard unix password manager for managing secrets that should not be in plaintext.

Optional:

```bash
# if you're doing PostgreSQL database development and need psql command
sudo apt-get install postgresql-client

# if you're doing your own software builds (instead of using binaries)
apt-get -y -qq install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libxml2-dev xz-utils tk-dev libxmlsec1-dev libreadline-dev libffi-dev libbz2-dev liblzma-dev llvm
```

### One-time setup of dotfiles

After installing `curl`, `git`, `just`, `asdf`, and `pass` you need to bootstrap the *dotfiles* (configuration files). We use [chezmoi](https://www.chezmoi.io/) with templates to manage our dotfiles across multiple diverse machines, securely. 

```bash
export CHEZMOI_CONF=~/.config/chezmoi/chezmoi.toml
mkdir -p `dirname $CHEZMOI_CONF`
cat << EOF > $CHEZMOI_CONF
[data]
    [data.git.user]
        name = "Shahid N. Shah"
        email = "user@email.com"
    [data.git.credential.helper.cache]
        timeout = 2592000 # 30 days
EOF
# Personalize your $CHEZMOI_CONF
vi $CHEZMOI_CONF
sh -c "$(curl -fsLS git.io/chezmoi)" -- init --apply netspective-studios/home-creators
exit
# open a new shell continue with one-time setup of CLI
```

See [chezmoi.toml Example](dot_config/chezmoi/chezmoi.toml.example) to help understand the variables that can be set and used across chezmoi templates.

After initial setup, once you exit and re-enter your shell, `cd ~ && just (cmd)` will be equivalent to `homectl (cmd)`. `homectl` is one of many aliases defined by the auto-imported [~/.config/z4h-zshrc/aliases.auto.zshrc](dot_config/z4h-zshrc/aliases.auto.zshrc.tmpl) file.

### One-time setup of CLI

Netspective Studios projects prefer ZSH as the default shell with a few typical projects to create a flexible creator's sandbox. The chezmoi setup above will introduce the dotfiles that this section needs.

Run [ZSH for Humans](https://github.com/romkatv/zsh4humans#installation) (`z4h`) installer: 

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/romkatv/zsh4humans/v5/install)"
```

Configure `.zshrc` and then install typical asdf plugins:

```zsh
cat << 'EOF' >> ~/.zshrc
# Hook direnv into your shell.
z4h source ~/.asdf/asdf.sh
z4h source -- ~/.config/z4h-zshrc/*.auto.zshrc
eval "$(asdf exec direnv hook zsh)"
direnv() { asdf exec direnv "$@"; }
EOF
exit
# open a new shell
homectl setup setup-asdf-plugins-typical
```

Personalize `z4h` with your own preferences. Learn how to use `z4h` to [keep it up to date](https://github.com/romkatv/zsh4humans#updating) and understand the recommended [customization approach](https://github.com/romkatv/zsh4humans#customization).

Customize your aliases, functions in `~/.config/z4h-zshrc/*.auto.zshrc` -- basically, any file in your `~/.config/z4h-zshrc` directory that has the `*.auto.zshrc` extension will be automatically sourced by `z4h` into each shell.

## Routine Maintenance

On a daily, weekly, or monthly basis run:

```bash
homectl maintain
```

## Managed Git Repos (GitHub, GitLab, etc.) Tools

Please review the bundled [Managed Git](managed-git/README.md) and opinionated set of instructions and tools for managing code workspaces that depend on multiple repositories.

## Polyglot langues installation and version Management

Netspective Studios projects assume that `asdf` is being used for version management of programming languages (Java, Go, etc.) and runtime environments (Deno, NodeJS, Python, etc.) and `direnv` is being used for project-specific environments. 

When you ran `just setup-asdf-plugins-typical`, as part of `z4h` CLI setup, `deno` was installed by default.

For other languages, you can install them like this:

```bash
asdf plugin add nodejs
asdf plugin add python
asdf plugin add java
asdf plugin add golang
asdf plugin add julia
asdf plugin add haxe
asdf plugin add neko
asdf plugin add hugo

asdf install golang latest
asdf install nodejs latest
asdf install python latest
asdf install hugo latest
...

asdf current
```

## Data Engineering

Default data tools installed in `~/bin`:

* [Miller](https://github.com/johnkerl/miller) is like awk, sed, cut, join, and sort for name\-indexed data such as CSV, TSV, and tabular JSON.
* [daff](https://github.com/paulfitz/daff) library for comparing tables, producing a summary of their differences.

If you run `just setup-data-engr-enhanced` you also get:

* [csvtk](https://github.com/shenwei356/csvtk) is a cross-platform, efficient and practical CSV/TSV toolkit in Golang.
* [xsv](https://github.com/BurntSushi/xsv) is a fast CSV command line toolkit written in Rust.
* [OctoSQL](https://github.com/cube2222/octosql) is a query tool that allows you to join, analyse and transform data from multiple databases and file formats using SQL.
* [q](http://harelba.github.io/q/) - Run SQL directly on CSV or TSV files.
* [Dasel](https://github.com/TomWright/dasel) jq/yq for JSON, YAML, TOML, XML and CSV with zero runtime dependencies.

TODO (others to consider):

* A list of command line tools for manipulating structured text data is available at https://github.com/dbohdan/structured-text-tools
* [eBay's TSV Utilities](https://github.com/eBay/tsv-utils): Command line tools for large, tabular data files. Filtering, statistics, sampling, joins and more. 
* [pgLoader](https://pgloader.io/) can either load data from files, such as CSV or Fixed-File Format; or migrate a whole database to PostgreSQL
  
## Beneficial Add-ons

### gitui terminal-ui for Git

[GitUI](https://github.com/extrawurst/gitui) provides you with the comfort of a git GUI but right in your terminal. Per their website, "this tool does not fully substitute the `git` CLI, however both tools work well in tandem". It can be installed using:

```bash
asdf plugin add gitui https://github.com/looztra/asdf-gitui
asdf install gitui latest
asdf global gitui latest
```

#### Important configuration management tools

This project's dotfiles automatically assume [asdf](https://asdf-vm.com/) is being used to manage multiple runtime versions with a single CLI tool. Per their documentation:

> asdf is a CLI tool that can manage multiple language runtime versions on a per-project basis. It is like gvm, nvm, rbenv & pyenv (and more) all in one! Simply install your language's plugin!

We use `asdf` to manage almost all languages and utilities so that they can be easily installed and, more importantly, support multiple versions simultaneously. For example, we heavily use `Deno` for multiple projects but each project might have a different version required. `asdf` supports global, per session, and per project (directory) [version configuration strategy](https://asdf-vm.com/#/core-configuration?id=tool-versions).

This project's dotfiles automatically setup [direnv](https://asdf-vm.com/) to encourage usage of environment variables with more flexibility. Per their documentation:

> direnv is an extension for your shell. It augments existing shells with a new feature that can load and unload environment variables depending on the current directory.

We use `direnv` and `.envrc` files to manage environments on a [per-directory](https://www.tecmint.com/direnv-manage-environment-variables-in-linux/) (per-project and descendant directories) basis. `direnv` can be used to [manage secrets](https://www.youtube.com/watch?v=x3p-28PajJY) as well as non-secret configurations. Many other [development automation techniques](http://www.futurile.net/2016/02/03/automating-environment-setup-with-direnv/) are possible.

`asdf` has [centrally managed plugins](https://asdf-vm.com/#/plugins-all) for many languages and runtimes and there are even more [contributed plugins](https://github.com/search?q=asdf) for additional languages and runtimes.

There are many [direnv YouTube videos](https://www.youtube.com/results?search_query=direnv) and [asdf videos](https://www.youtube.com/watch?v=r6qLQgq2vGk) worth watching to get familar with the capabilities.