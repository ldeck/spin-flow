# git spin-flow
Thinking through a practical team approach to git *flow.

## Background ##

Lots has been written about whether GitFlow, [proposed by Vincent Driessen](https://nvie.com/posts/a-successful-git-branching-model/), is beneficial or [considered harmful](https://www.endoflineblog.com/gitflow-considered-harmful). I won't repeat the discussion here.

There are *LOTS* of Git flows available, each with a variation on the theme and discussion points to cover. Here's a non-exhaustive list to digest.

* <https://softwareengineering.stackexchange.com/questions/347525/should-a-release-branch-or-the-master-branch-be-tagged-when-the-gitflow-is-used>
* <https://stackoverflow.com/questions/47739701/gitflow-merge-to-master-first-or-after-prod-release>
* <https://blogs.tensult.com/2019/02/12/version-controlling-using-git-flow-tags/>
* <https://georgestocker.com/2020/03/04/please-stop-recommending-git-flow/>
* <https://brightinventions.pl/blog/how-do-we-use-git>
* <https://dev.to/scottshipp/war-of-the-git-flows-3ec2>
* <https://docs.microsoft.com/en-us/azure/devops/learn/devops-at-microsoft/use-git-microsoft>
* <https://www.nomachetejuggling.com/2017/04/09/a-different-branching-strategy/>

Long story short, the conceptual problems I have with GitFlow are these, at the least:

* Multiple long-lived branches: master and develop. i.e., one of them is redundant.
* Requires setup of the repository to work (seems like tight coupling).
* Reliance on a branch rather than tags to track production deployed code.

In the end, my suggested git spin-flow takes concepts from all the discussions to arrive at something that seems to make sense, at least to me at this time.

## Why spin? ##

When thinking about a name that could helpfully describe this flow, [funnel-web spiders](https://www.nationalgeographic.org/photo/burrow-spider/) came to mind. How do spiders manage their webs? They spin them up, out, in, and so forth. These directional prepositions might be useful to describe the concepts at play. See [here](https://linguapress.com/grammar/prepositions-adverbs.htm) for further explanation of prepositions and adverbs.


[![Funnel-Web Spider courtesy National Geograhic](https://media.nationalgeographic.org/assets/photos/000/309/30980.jpg "Keep your distance")](https://www.nationalgeographic.org/photo/burrow-spider/#funnel-web-spider)

## spin-flow highlights ##

* A single long-lived branch: master (not guaranteed to be shippable)
* A series of short-lived branches (feature, release, hotfix, ops, wip)
* Tagged production releases
* Rebase + Merge —no-ff (as per [oneflow’s option #3 for finishing features](https://www.endoflineblog.com/oneflow-a-git-branching-model-and-workflow#finishing-a-feature-branch))
* Hot fixes against a production release tag (cherry-picked preferably), or the current release branch
* Avoids squash commits

## spin-flow concepts ##

### PRIMARY BRANCH ==> next release candidate ###

The *primary branch* is not guaranteed to be a shippable. It represents the latest amalgamation of features that have been developed, reviewed and tested for the *next release*. There is no guarantee that these features have gone through rigourous QA testing. Neither is it guaranteed they have been released to production.

    master
    0—0—0------0—0

### FEATURE DEVELOPMENT ###

Feature development occurs outside the *primary branch* to enable fast switching when needed between tasks. Features should be small, self-contained units, able to be published quickly. Automated testing should be used to ensure it continuous to behave as desired after being incorporated into the primary branch. Commits to master directly may be optionally disabled to enforce this process so the team isn't blocked by an untested commit.

Feature development consists of three spun stages: `spin off, spin out, spin into`. i.e., branch, push, merge. Inbetween out and into spins are automated testing, code reviews, and revision. These ought to be incorporated into your CI/CD pipelines.

#### feature spin-off (branch) ####

Feature development typically begins by branching off the *primary branch* in order to contribute a likely requested feature to the *next release*.

    master
    0—0—0------0—0
         \
          0—0—0—0
          feature-a


    $ git checkout master
    $ git pull
    $ git checkout -b feature/feature-a

#### feature spin-out (push) ####

When the feature is ready to be published to the wider team it should be pushed out to the central repository. A merge/pull request should be raised to request that the feature be incorporated into the *next release*.

    git push -u origin feature/feature-a

CI/CD pipelines should include automated testing to ensure it's a feature ready to be reviewed.

    git checkout feature/feature-a
    # run automated tests
    # might deploy the app to a sandbox environment for deeper testing

#### feature spin-into (merge) ####

Once a feature has been tested and accepted, it may now be incorporated into the *next release*. Rebasing against the *primary branch* prior to incorporating it into the candidate *next release* results in keeping the history of the repository clean.

    master
    0—0—0------0—0—————0
                 \     /
                  0—0—0
                  feature-a


    $ git rebase -i master
    # re-run automated tests
    $ git checkout master
    $ git merge --no-ff feature/feature-a
    $ git push origin master
    $ git branch -d feature/feature-a

## Contributions ##

Contributions are more than welcome. If you have suggestions on how to simplify and solidify the process further that'd be fantastic.

## Thanks ##

Many thanks to the many who have provided a detailed explanation and discussion on their chosen workflow or commentary on limitations they see in a suggested workflow.
