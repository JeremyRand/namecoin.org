---
layout: post
title: "rbm on Cirrus CI"
author: Jeremy Rand
tags: [News]
---

rbm (Reproducible Build Manager) is a quite nice system for reproducible builds.  rbm was originally created by Nicolas Vigier from the Tor Browser Team as a replacement for Gitian (which was previously used by Tor Browser and is still used by Bitcoin Core).  rbm has since been adopted by other projects like Ricochet and Namecoin (in particular, we use rbm for building ncdns).  Recently, I've been doing some mad science experiments with running rbm on Cirrus CI infrastructure.

## Why use Cirrus CI for reproducible builds?

Being able to run rbm as part of a CI workflow brings several benefits:

1. We can test PR's to make sure they build properly, without needing a developer machine to do the build.
2. We can upload nightly build binaries, and binaries built from PR's, again without needing a developer machine.
3. We can use the CI service as a co-signer for reproducible release builds.

The third may merit some further explanation, since you may be wondering why anyone should trust Cirrus CI to sign our builds.  The underlying principle of reproducible builds is **not** that any particular co-signer is supposed to be trustworthy.  Rather, the co-signers should be **diverse**, such that an event that compromises a subset of the co-signers is not likely to compromise the remainder.  For example, it is desirable for different co-signers to run different OS distributions, not because any of those distributions are trustworthy, but simply because a compromise of Debian signing keys is unlikely to also result in a compromise of Fedora signing keys (or vice versa).  This explains why it's desirable to use Cirrus CI as a co-signer: the circumstances that would cause Cirrus CI infrastructure to be compromised are quite different from the circumstances that would cause the Namecoin developers' build machines to be compromised; thus using Cirrus CI as a co-signer improves diversity, which by extension improves security.

Why did I pick Cirrus CI instead of other CI services like GitLab, GitHub Actions, or Microsoft Azure Pipelines?  Cirrus made sense because Bitcoin Core already uses it, meaning that Namecoin already uses it too.  However, it should be noted that there's no good reason to stick to only one CI service here; more CI services means more diversity, and thus more security.  I intend to try porting this work to other CI service in the future.

## Running rbm at all

Cirrus typically runs all GNU/Linux builds inside a Docker container.  Meanwhile, rbm uses `runc` to start containers of its own.  [As Hitler wants us to know](https://www.youtube.com/watch?v=PivpCKEiQOQ), running a container inside a container is an easy way to lose World War II.  However, Cirrus does have a minimally documented build mode called `docker_builder`.  This is described in the documentation as a mechanism to build custom Docker containers so that you can then use those containers to run your actual build in the normal way inside Docker.  However, the `docker_builder` mode is actually just a standard Ubuntu VM, and it works totally fine to run an rbm build in this environment without ever touching Docker [1].

## Time limits: Caching

Cirrus has a default time limit per task of 1 hour.  This can be increased to a maximum of 2 hours via YML configuration, but 2 hours is still too short a duration to build Tor Browser (or ncdns).  However, there are some hacks we can use here.  First off, we can instruct Cirrus to cache the `out` directory that rbm creates during a build.  This means that once we build a specific project (e.g. GCC or the Go compiler), it won't have to be rebuilt unless its version or dependencies are bumped.  We can also cache the `git_clones` directory, which means that only new Git commits must be downloaded, rather than cloning entire repositories on each build.

## Time limits: Splitting by Project

Once we're caching the outputs of projects, we can instruct Cirrus to split the build into several tasks, each of which builds only a subset of projects.  For example, Namecoin currently builds GCC, the Go compiler, and a few Go libraries in one task, while we build ncdns in another task.  I wrote a Bash script that automates construction of a Cirrus YML config that contains many tasks that all run rbm and are executed sequentially.  (Maybe I could scrap the Bash script in favor of smarter YML usage?  Looking into that is for another day.)

## Time limits: The Clang Problem

A few of Tor Browser's rbm projects take longer than 2 hours to build.  The good news is that none of them are necessary to build any Namecoin projects for GNU/Linux or Windows targets.  The bad news is that one of them (Clang) is needed to build Namecoin for macOS.  And of course, since Namecoin is a good neighbor, I want to get this upstreamed to Tor, so Tor's problems are my problems.  I considered a few options here:

1. Try to refactor the affected projects so that they can be split into smaller projects that each take less build time.  I rejected this approach because (1) it would probably be fairly invasive to upstream, and (2) it would probably slow down builds that aren't running on Cirrus (there is a nontrivial per-project overhead in rbm).
2. Checkpoint the build containers with [CRIU](https://www.criu.org/) ("Checkpoint/Restore In Userspace"; basically the container equivalent of the "hibernate" feature that operating systems have on bare-metal), so we can split their execution into multiple Cirrus tasks.  This seemed like a good option since runc has built-in support for CRIU, until Nicolas from Tor gave me a heads up that rbm was soon going to replace runc with something built in-house, which does not support CRIU.
3. Send the SIGINT signal to a build container to interrupt its build script, then cache the container's filesystem, and then run the build script again in a subsequent Cirrus task.  This is conceptually similar to CRIU, but it abuses the fact that most build systems (e.g. GNU Make) are smart enough to not repeat already-completed steps if you run them a 2nd time.  In contrast, CRIU is designed to handle the more general case where a program inside a container isn't that smart -- we do not need that level of robustness here, and sending SIGINT to everything inside a build container is a lot simpler than using CRIU with a container system that isn't designed to support it.

Option 3 is what I decided to go with.  I initially wasn't sure how to send SIGINT to a container, but some brief experiments with `pgrep` on the host OS showed that all processes inside the rbm container were visible from the host OS, including their command-line metadata.  Since rbm always launches a build inside the container by running a script called `build`, that means we can identify the host-side process ID of the build script inside the container by running `pgrep -f '\./build'`.  Furthermore, I found [a Stack Exchange answer by Russell Davis](https://unix.stackexchange.com/a/299198) on obtaining all of the descendent process ID's (e.g. the `make` and `gcc` processes); this also works from the host OS.

From this, I wrote a quick Bash script (which runs on the host OS) that first sets a flag (via `touch`ing a file) indicating that the build was interrupted, and proceeds to send SIGINT to the build script and all of its descendent processes inside the rbm container.  Next, I patched rbm so that when the build script exits, rbm checks if that flag is set.  If it was set, then rbm saves a copy of the container's filesystem before deleting the container.  The next time rbm tries to build that particular project hash, it checks whether it had previously saved a filesystem for that project hash; if so, it restores the filesystem instead of creating a clean container image.

Finally, I had to slightly patch the build script for the Clang toolchain projects.  This was necessary because although GNU Make is smart enough to not get confused by being run twice in a row, Tor's Bash script for building Clang is not quite that smart.  I didn't need to make the Bash build script tolerate being restarted at arbitrary points; the only place where it was going to restart from was the `make` command.  So, I simply made the build script set a flag (again, by `touch`ing a file) right before it ran the `make` command; this gets saved along with the rest of the container's filesystem if the build gets interrupted with SIGINT.  If the build script finds that this flag is set at the beginning of the script, it skips ahead in the script straight to the `make` command.  Sure, it's a little hacky, but it's dead simple and generally noninvasive.

## Results

Namecoin's rbm build system now runs inside Cirrus CI, meaning that we can use Cirrus as a reproducible build co-signer.  In addition, I set up a Cirrus cron task that checks daily for dependencies whose versions we can bump, and automatically submits a GitHub PR when it finds some.  Since Cirrus builds our GitHub PR's, this allows us to automatically test whether the bumped dependencies still build without errors, saving us even more time.

I gave a presentation on this work at Tor Demo Day; the response from the Tor developers was enthusiastic.  (My understanding is that Tor intends to post a video of the presentation.)  I intend to send these patches upstream to Tor once Namecoin has battle-tested them for a few months.

In the field of reproducible builds, security derives from diversity.  Adding the diversity of public CI infrastructure to the Namecoin and Tor reproducible build systems yields a major security bump.  I'm looking forward to getting this upstreamed, and maybe adding additional CI services like GitLab and GitHub Actions to the mix in the future.

[1] Yes, I know that Hitler was referring to a container inside a VM, which means we're still violating Hitler's advice.  Quiet, you.
