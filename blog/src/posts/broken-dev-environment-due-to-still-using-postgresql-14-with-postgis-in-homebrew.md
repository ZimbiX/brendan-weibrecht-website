# Broken dev environment due to still using PostgreSQL 14 with PostGIS in Homebrew

2025-09-29

## Intro

I had a bit of a headache to deal with today, due to some breakage from upgrading Homebrew packages. After the update, when I ran RSpec, I'd see:

```
ActiveRecord::StatementInvalid:
  PG::UndefinedFile: ERROR:  could not access file "$libdir/postgis-3": No such file or directory
```

## Context

I'm working on a project that's still using [PostgreSQL](https://www.postgresql.org/) 14 (the latest is 18). It also uses a PostgreSQL extension called [PostGIS](https://postgis.net/):

> PostGIS extends the capabilities of the PostgreSQL relational database by adding support for storing, indexing, and querying geospatial data.

For developer machines, the default setup is that both are installed through Homebrew currently.

It would probably be a lot of effort to upgrade PostgreSQL, and there isn't the capacity to do so at the moment. PostgreSQL 14 is [supported until 2026-11-12](https://endoflife.date/postgresql), which is far enough away that it's not a concern at the moment.

## Breakage

I'd last updated just a couple of days ago, and everything worked fine. This time, the Homebrew upgrade logging noted:

```
==> Upgrading 6 outdated packages:
[...]
postgis 3.6.0 -> 3.6.0_1
```

Who would've thought that'd be a breaking change? TIL

## Investigation

I tried a bunch of things I found online before thinking too deeply about the problem, but none of these worked:

```
brew services restart postgresql@14
psql -c 'ALTER EXTENSION postgis UPDATE;'
psql -c "ALTER EXTENSION postgis UPDATE to '3.6.0';"
psql -c 'SELECT postgis_extensions_upgrade();'
psql -c 'SELECT postgis_full_version();'
postgis enable
```

Someone suggested we might require a specific version of PostGIS. But the latest PostGIS does [still support PostgreSQL 14 (PostgreSQL 12 - 18beta3)](https://postgis.net/2025/09/PostGIS-3.6.0/).

Eventually, I looked at the [`postgis` Homebrew formula source](https://github.com/Homebrew/homebrew-core/blob/main/Formula/p/postgis.rb) more closely.

## The cause

I noticed that the `postgis` Homebrew formula was only installing PostGIS into the versions of PostgreSQL that it mentions as dependencies. Those dependencies had just been changed - [from 14 and 17, to just 17 and 18](https://github.com/Homebrew/homebrew-core/commit/7897963459566ad30f1e3b75d411d2310cd6397d).

I could confirm this using `find /opt/homebrew -iregex '.*postgis.*' 2>/dev/null` and saw that a bunch of files existed in directories for those versions but not 14.

Well, that sucks.

On the linked PR, I asked why they'd make that change, and [graciously received a reply from the head of Homebrew](https://github.com/Homebrew/homebrew-core/pull/245698#discussion_r2386858371):

> More versions means longer builds and our CI time is constrained.

Fair enough, I guess. It's just confusing that a `postgresql@14` package is offered, but now not supported by the PostgreSQL extension packages.

The Arch Linux repositories only keep the [latest version of each package](https://archlinux.org/packages/?q=postgresql), [demoting older versions into the Arch User Repository](https://aur.archlinux.org/packages?O=0&SeB=n&K=postgresql&outdated=&SB=n&SO=d&PP=50&submit=Go). Requiring a specific version and getting it from the Arch repos is unwise; and doing so from Homebrew is probably similarly unwise.

## Solution

I have a (somewhat dodgy) solution working, based on [Homebrew's FAQ guide for editing a formula](https://docs.brew.sh/FAQ#can-i-edit-formulae-myself):

- Add `export HOMEBREW_NO_INSTALL_FROM_API=1` to your `~/.zshrc` and open a new shell
- `brew tap --force homebrew/core`
- `brew edit postgis`
- Replace the lines beginning with `depends_on "postgresql@` with just this line: `depends_on "postgresql@14" => [:build, :test]`
- `brew reinstall --build-from-source --verbose --debug postgis`
- `psql -c 'ALTER EXTENSION postgis UPDATE;'`. I did this also before testing it - I'm not sure if that was required

Notes from the linked guide:

> You don’t have to submit modifications back to homebrew/core, just edit the formula to what you personally need and `brew install <formula>`. As a bonus, `brew update` will merge your changes with upstream so you can still keep the formula up-to-date with your personal modifications!
>
> Note that if you are editing a core formula or cask you must set `HOMEBREW_NO_INSTALL_FROM_API=1` before using `brew install` or `brew update` otherwise they will ignore your local changes and default to the API.
>
> To undo all changes you have made to any of Homebrew’s repositories, run `brew update-reset`. It will revert to the upstream state on all Homebrew’s repositories.

## Alternatives

I'm not sure these steps are suitable for inclusion in our project's setup script. I think we should strongly consider switching how we install PostgreSQL by default. Maybe instead creating a Homebrew tap, or using Docker Compose, asdf, or mise.

### Tap

From https://docs.brew.sh/Versions:

> Homebrew’s versions should not be used to “pin” formulae to your personal requirements. You should instead create your own tap for formulae you or your organisation wish to control the versioning of, or those that do not meet the above standards. Software that has regular API or ABI breaking releases still needs to meet all the above requirements; that a brew upgrade has broken something for you is not an argument for us to add and maintain a formula for you.

We could create a tap. But then it'd be a manual process to incorporate updates into it from the upstream formula, which seems like unnecessary busywork.

### Docker Compose

I'd used PostgreSQL through Docker Compose for years before this at GreenSync, and found that to be excellent; allowing easy setup, different versions between services (as long the database of each project used a unique port), and consistent use of the db between local dev and CI. But that was on Linux, where I didn't have the performance overhead of needing a virtual machine to run Docker images. Most of the others in the company used macOS though, and rarely was there a complaint about db performance.

Frustratingly, I've recently been forced to start using macOS for the first time, required for this work; after coming from using Linux as my primary OS since ~2012.

In my opinion, Homebrew is a poor subtitute for Arch Linux's [Pacman](https://wiki.archlinux.org/title/Pacman) package manager. Slow servers and a lack of parallel downloading make installs/upgrades a pain when there is a lot of work for it to do. The `postgis` package has many dependencies, and the time it takes to install them all makes that by far the slowest part of our project's setup script, which, from memory, resultingly takes ~30 minutes on a new machine. It would be nice to avoid this slowdown.

### asdf

We do already use [asdf](https://asdf-vm.com/) for installing Ruby, NodeJS, and a few of their dependencies. So we could potentially use it for PostgreSQL as well. I briefly used a few different versions of PostgreSQL through it recently (while I was debugging [this Rails `structure.sql` issue](https://github.com/rails/rails/issues/55509)), and found it relatively straightforward. Using it with PostGIS might be another story though, but there is an [`asdf-postgis` plugin](https://github.com/knu/asdf-postgis) to look into.

Sidenote: asdf lacks an equivalent of [rbenv](https://github.com/rbenv/rbenv)'s `rbenv shell 3.4.6`, which sets the version of Ruby in just the current shell - so I implemented that in an [asdf wrapper shell function](https://github.com/ZimbiX/dotfiles/blob/master/asdf/asdf-wrapper.zsh). Even so, I do miss rbenv's UX sometimes.

### mise

My colleague Mario showed me [mise](https://github.com/jdx/mise), which looks pretty cool. Compared with asdf, mise has [better UX, performance, and security](https://mise.jdx.dev/dev-tools/comparison-to-asdf.html#ux). I would have switched to mise already if I hadn't only just learned asdf, and wasn't already dealing with the high level of learning required in starting at a new company.

From Mise's [tools registry](https://mise.jdx.dev/registry.html#tools), it looks to use the same plugins as asdf for [postgres](https://github.com/mise-plugins/mise-postgres) and [postgis](https://github.com/mise-plugins/mise-postgis) (but forked with basically no changes). That's handy reuse.

### Something else?

If you have a better idea, let me know! =)
