# git spin-flow
Thinking through a practical team approach to git *flow.

## Background ##

Lots has been written about whether [GitFlow](https://github.com/nvie/gitflow), [proposed by Vincent Driessen](https://nvie.com/posts/a-successful-git-branching-model/), is beneficial or [considered harmful](https://www.endoflineblog.com/gitflow-considered-harmful). I won't repeat the discussion here.

There are *LOTS* of Git flows available, each with a variation on the theme and discussion points to cover. Here's a non-exhaustive list to digest.

* <https://softwareengineering.stackexchange.com/questions/347525/should-a-release-branch-or-the-master-branch-be-tagged-when-the-gitflow-is-used>
* <https://stackoverflow.com/questions/47739701/gitflow-merge-to-master-first-or-after-prod-release>
* <https://blogs.tensult.com/2019/02/12/version-controlling-using-git-flow-tags/>
* <https://georgestocker.com/2020/03/04/please-stop-recommending-git-flow/>
* <https://brightinventions.pl/blog/how-do-we-use-git>
* <https://dev.to/scottshipp/war-of-the-git-flows-3ec2>
* <https://medium.com/@brunoluiz/still-using-gitflow-what-about-a-simpler-alternative-74aa9a46b9a3>
* <https://docs.microsoft.com/en-us/azure/devops/learn/devops-at-microsoft/use-git-microsoft>
* <https://www.nomachetejuggling.com/2017/04/09/a-different-branching-strategy/>
* <https://datasift.github.io/gitflow/GitFlowForGitHub.html>
* <https://blogs.tensult.com/2019/02/12/version-controlling-using-git-flow-tags/>

Long story short, the conceptual problems I have with GitFlow are these, at the least:

* Multiple long-lived branches: master and develop. i.e., one of them is redundant.
* Requires setup of the repository to work (seems like tight coupling).
* Reliance on a branch rather than tags to track production deployed code.
* And so on.

In the end, I am weaving together something that takes concepts from all the discussions to arrive at something that seems to make sense, at least to me at this time, and which aims at being simple. This is a work in progress.

## Why spin? ##

When thinking about a name that could helpfully describe this flow, [funnel-web spiders](https://www.nationalgeographic.org/photo/burrow-spider/) came to mind. How do spiders manage their webs? They spin them up, out, in, and so forth. These directional prepositions might be useful to describe the concepts at play. See [here](https://linguapress.com/grammar/prepositions-adverbs.htm) for further explanation of prepositions and adverbs.


[![Funnel-Web Spider courtesy National Geograhic](https://media.nationalgeographic.org/assets/photos/000/309/30980.jpg "Keep your distance")](https://www.nationalgeographic.org/photo/burrow-spider/#funnel-web-spider)

## spin-flow highlights ##

* A single long-lived branch: master (not guaranteed to be shippable)
* A series of short-lived branches (feature, hotfix, ops, wip)
* Choice: release short-lived branch OR tag next candidate
* Tagged release candidates and production releases
* Rebase + Merge —no-ff (as per [oneflow’s option #3 for finishing features](https://www.endoflineblog.com/oneflow-a-git-branching-model-and-workflow#finishing-a-feature-branch))
* Hot fixes against a production release tag (cherry-picked preferably), or the current release branch
* Avoids squash commits

## spin-flow concepts ##

### PRIMARY BRANCH ==> next release candidate ###

The *primary branch* is not guaranteed to be a shippable version of the product. It represents the latest amalgamation of features that have been developed, reviewed and tested for the *next release*. There is no guarantee that these features have gone through rigourous QA testing. Neither is it guaranteed they have been released to production.

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

### RELEASES ###

Releases are typically either planned OR identified from a set of features that are deemed ready for pushing out.

The end result is an identifiable tagged release. The means of arriving at said tag could be from a candidate tag or release branch.

The choice to create a release branch or to use a release candidate tag seems to boil down to whether or not you intend to amend the release over time or not. Amending over time would assume additional commits are cherry-picked into the release candidate branch. These additions might include release notes, or late arrival features deemed important, or even hotfixes for bugs discovered in higher environments. The smaller the set of features and additions, the faster the release can be tested and pushed to production (i.e., fast feedback principles apply).

#### RELEASE BRANCHING ####

##### release spin-off (branch) #####

We begin by creating a release branch from the either the primary branch OR the previous release. Microsoft's model of always cherry-picking features for inclusion into a release suits the latter of these.

    master
    0—0—0------0—0—————0---0
                 \
                  0—0
                  release-1.2.3

    $ git checkout -b release/1.2.3 <ref>

##### release spin-out (push) #####

This step may occur multiple times as the release is amended so that QA may sign off on each amendment. This likely involves deployment to a staging environment.

##### release spin-into (tag)  #####

Closing out a release involves tagging the tip of the release and preserving it by merging it into the *primary branch*. If you're always cherry-picking into the release branch, the merge should be a no-op.

    master
    0—0—0------0—0—————0
                 \     /
                  0—0—0
                  release-1.2.3


    $ git checkout release/1.2.3
    $ git tag 1.2.3
    $ git checkout master
    $ git merge release/1.2.3
    $ git push --tags origin master
    $ git branch -d release/1.2.3
    $ git push origin :release/1.2.3

You'll notice that there is no real distinction at this point between a feature being spun into the primary branch or a release. The primary difference is quality assurance.

#### RELEASE TAGGING ####

Using a tag to identify a release candidates might be appropriate where release documentation is external to the repository (shudder the thought; here's looking at you Confluence).

##### release spin-off (tag candidate) #####

    master
    0—0—0------0—0—————0
                 \
                  0
                  candidate/1.2.3

    $ git tag candidate/1.2.3 <ref>

##### release spin-out (stage) #####

Perform validation.

##### release spin-into (tag release) #####

If the candidate passes validation, tag the release and remove the candidate tag. If the candidate fails validation, remove the tag which allows a new candidate to be identified.

    master
    0—0—0------0—0—————0
                 \
                  0
                  release/1.2.3

    $ git tag release/1.2.3 <ref>
    $ git tag -d candidate/1.2.3 <ref>
    $ git push --delete origin candidate/1.2.3

#### HOTFIXES ####

Hotfixes are like features but are aimed at rectifying another feature was included in another release or release candidate.

#### hotfix spin-off (branch) ####

Hotfix development typically begins by branching off a release in order to contribute a likely requested hotfix to the *next minor release*.

    master
    0—0—0------0—0
         \
          0—0—0—0
          hotfix-a


    $ git checkout -b hotfix/hotfix-a release/1.2.3

#### hotfix spin-out (push) ####

When the hotfix is ready to be published to the wider team it should be pushed out to the central repository. A merge/pull request should be raised to request that the hotfix be incorporated into the *next release*.

    git push -u origin hotfix/hotfix-a

CI/CD pipelines should include automated testing to ensure it's a hotfix ready to be reviewed.

    git checkout hotfix/hotfix-a
    # run automated tests
    # might deploy the app to a sandbox environment for deeper testing

#### hotfix spin-into (merge) ####

Once a hotfix has been tested and accepted, it may now be incorporated into the *next minor release*.

    master
    0—0—0------0—0—————0
        \             /
         0—0—--------0
         hotfix-a


    $ git checkout hotfix/1.2.3
    $ git tag 1.2.3
    # re-run automated tests
    $ git checkout master
    $ git merge --no-ff hotfix/hotfix-a
    $ git push --tags origin master
    $ git branch -d hotfix/hotfix-a


## Contributions ##

Contributions are more than welcome. If you have suggestions on how to simplify and solidify the process further that'd be fantastic.

Thanks in particular to Adam Ruka (@skinny85) for articulating his oneflow so carefully.

## Thanks ##

Many thanks to the many who have provided a detailed explanation and discussion on their chosen workflow or commentary on limitations they see in a suggested workflow.
