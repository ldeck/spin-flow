# git spin-flow
Thinking out loud through a practical team approach to git *flow.

## Background ##

Lots has been written about whether [GitFlow](https://github.com/nvie/gitflow), [proposed by Vincent Driessen](https://nvie.com/posts/a-successful-git-branching-model/), is beneficial or [considered harmful](https://www.endoflineblog.com/gitflow-considered-harmful). I don't aim to repeat the discussion here.

There are *LOTS* of Git flows available, each with a variation on the theme and discussion points to cover. Here's a non-exhaustive list to digest for your own investigation.

* <https://softwareengineering.stackexchange.com/questions/347525/should-a-release-branch-or-the-master-branch-be-tagged-when-the-gitflow-is-used>
* <https://stackoverflow.com/questions/47739701/gitflow-merge-to-master-first-or-after-prod-release>
* <https://blogs.tensult.com/2019/02/12/version-controlling-using-git-flow-tags/>
* <https://georgestocker.com/2020/03/04/please-stop-recommending-git-flow/>
* <https://brightinventions.pl/blog/how-do-we-use-git>
* <https://dev.to/scottshipp/war-of-the-git-flows-3ec2>
* <https://medium.com/@brunoluiz/still-using-gitflow-what-about-a-simpler-alternative-74aa9a46b9a3>
* <https://www.nomachetejuggling.com/2017/04/09/a-different-branching-strategy/>
* <https://datasift.github.io/gitflow/GitFlowForGitHub.html>
* <https://blogs.tensult.com/2019/02/12/version-controlling-using-git-flow-tags/>
* <https://www.martindroessler.de/blog/index.php?/archives/83-Establishing-a-semi-automated-gitflowversioning-process-with-a-maven-project.html>

Official guides:
* <https://www.git-tower.com/learn/git/ebook/en/command-line/advanced-topics/git-flow/>
* https://guides.github.com/introduction/flow/
* <https://docs.gitlab.com/ee/topics/gitlab_flow.html>
* <https://docs.microsoft.com/en-us/azure/devops/learn/devops-at-microsoft/use-git-microsoft>
* <https://www.atlassian.com/blog/git/simple-git-workflow-is-simple>

Semantic versioning (related topic) and continous delivery/deployment:
* <https://semver.org>
* <https://www.cloudbees.com/blog/apache-maven-continuous-deliverydeployment-devoptics-teams-approach>
* <https://viesure.io/automating-semantic-versioning-with-maven/>
* <https://github.com/semantic-release/semantic-release>
* <https://levelup.gitconnected.com/semantic-versioning-and-release-automation-on-gitlab-9ba16af0c21>
* <https://medium.com/trendyol-tech/semantic-versioning-and-gitlab-6bcd1e07c0b0>

Gitlab tagging / release issues and examples:
* <https://gitlab.com/gitlab-org/gitlab-foss/-/issues/23894>
* <https://gitlab.com/gitlab-org/gitlab/-/issues/16290>
* <https://github.com/viesure/blog-gitflow-maven/blob/master/.gitlab-ci.yml>
* <https://docs.gitlab.com/ee/ci/yaml/#rules>
* <https://stackoverflow.com/questions/47651769/gitlab-ci-run-build-job-when-manual-or-when-master-only>
* <https://docs.gitlab.com/ee/user/project/push_options.html#push-options-for-gitlab-cicd>

Long story short, the conceptual problems I have with GitFlow are these, at the least:

* Multiple long-lived branches: master and develop. One of them is likely redundant.
* Requires setup of the repository to work (seems like tight coupling).
* Reliance on a branch rather than tags to track production deployed code.
* Assumes only a single deployment target.
* And so on.

In the end, I am weaving together something that takes concepts from all the discussions to arrive at something that seems to make sense, at least to me at this time, and which aims at being simple and importantly successfully reliable. This _is_ a work in progress, but contributions and tools to facilitate it are very welcome!

## Why spin? ##

Essentially I'm spinning together a flow which, somewhat like a spider spinning its web, takes advantage of its surroundings. The Sydney [funnel-web spider](https://www.nationalgeographic.org/photo/burrow-spider/) comes to mind. It's deadly, but careful. Spiders spin their webs up, out, in, and so forth. These directional prepositions could be useful to describe the concepts and actions applicable. See [here](https://linguapress.com/grammar/prepositions-adverbs.htm) for further explanation of prepositions and adverbs.

[![Funnel-Web Spider courtesy National Geograhic](https://media.nationalgeographic.org/assets/photos/000/309/30980.jpg "Keep your distance")](https://www.nationalgeographic.org/photo/burrow-spider/#funnel-web-spider)

## spin-flow highlights ##

* A single long-lived branch: master (not guaranteed to be shippable)
* A series of short-lived branches (feature, hotfix, candidate, etc)
* Tagged releases
* Rebase + Merge —no-ff (as per [oneflow’s option #3 for finishing features](https://www.endoflineblog.com/oneflow-a-git-branching-model-and-workflow#finishing-a-feature-branch))
* Hot fixes against a production release tag (cherry-picked preferably), or the current release branch
* Avoids squash commits

## spin-flow concepts ##

### SNAPSHOT BRANCH ==> potential release candidate ###

The *snapshot branch* (typically master) is not guaranteed to be a shippable version of the product. It represents instead the latest amalgamation of features that have been developed, reviewed and accepted for the *next planned release*. There is no guarantee that these features have gone through rigourous QA testing. Neither is it guaranteed they have been released to production. It is a continous recipient of feature delivered changes and represents the teams efforts towards the next release.

    master
    0—0—0------0—0

The version of the software is always 1.x-SNAPSHOT. The release version is always yet to be determined.

### FEATURE DEVELOPMENT ###

Feature development occurs outside the *snapshot branch* to enable fast switching when needed between tasks. Features should be small, self-contained units, able to be published quickly. Automated testing should be used to ensure it continuous to behave as desired after being incorporated into the primary branch. Commits to master directly may be optionally disabled to enforce this process so the team isn't blocked by an untested commit.

Feature development consists of three spun stages: `spin off, spin out, spin into`. i.e., branch, push, merge. Inbetween out and into spins are any necessary automated testing, code reviews, and revision. These ought to be incorporated into your CI/CD pipelines to limit manual processes.

Versions typically remain untouched during feature development.

#### feature spin-off (branch) ####

Feature development typically begins by branching off the *snapshot branch* in order to contribute a likely requested feature to the *next candidate release*.

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

##### feature spin-down (discard) #####

There are times when a feature is deemed obsolete. This should delete the feature branch.

### CANDIDATE RELEASES ###

Candidate Releases are typically either planned OR identified from a set of features that are deemed ready for pushing out after more rigourous testing. The interval is determined by the team. Smaller increments lead to faster feedback and less big-bang testing.

The end result is an identifiable release tag. To get there, we start by branching from our *snapshot branch* and determining the target release version.

While the release is being tested, final release notes may be prepared (hopefully automated as much as possible).

If, for example, the previous release version was 1.2.0 the release candidate will begin as 1.3.0-SNAPSHOT.

#### CANDIDATE BRANCHING ####

##### candidate spin-off (branch) #####

We begin by creating a *candidate branch* from the the *current snapshot* branch head.

    master
    0—0—0------0—0—————0---0
                 \
                  0—0
                  candidate/release-1.2.0-SNAPSHOT

    $ git checkout -b candidate/1.2.0-SNAPSHOT <ref>

The branch ought to cause a deployment to a staging environment for rigourous testing.

##### candidate spin-out (tag) #####

Closing out a release involves tagging the verified candidate and deleting its branch.

    master
    0—0—0------0—0—————0
                 \
                  0—0—0
                      release/1.2.3


    $ git checkout candidate/1.2.0-SNAPSHOT
    $ git tag release/1.2.0
    $ git push --tags origin master
    $ git branch -d candidate/1.2.0-SNAPSHOT
    $ git push origin :candidate/1.2.0-SNAPSHOT

##### candidate spin-down (delete) #####

There are times when a candidate may be abandoned. The choices include:
- discarding the branch as if it never existed
- tagging the release candidate anyway, for versioning purposes, but not pushing to production


#### HOTFIXES ####

Hotfixes are like features but are aimed at rectifying another feature that was included in another release or release candidate.

#### hotfix spin-off (branch) ####

Hotfix development typically begins by branching off a release in order to contribute a likely requested hotfix to the *next patch release*.

        release/1.2.0
    0—0—0
         \
          0—0—0—0
          hotfix-a


    $ git checkout -b hotfix/hotfix-a release/1.2.0

#### hotfix spin-out (push) ####

When the hotfix is ready to be published to the wider team it should be pushed out to the central repository. A merge/pull request should be raised to request that the hotfix be incorporated into the *next release*.

    git push -u origin hotfix/hotfix-a

CI/CD pipelines should include automated testing to ensure it's a hotfix ready to be reviewed.

    git checkout hotfix/hotfix-a
    # run automated tests
    # might deploy the app to a sandbox environment for deeper testing

#### hotfix spin-into (merge) ####

A hotfix may be applicable for a *release candidate* currently being tested. In this case, the hotfix may be merged into the current *candidate* branch.
Once a hotfix has been tested and accepted, it may now be incorporated into the *next minor release*.

    master
    0—0—0------0—0—————0
       \
        \     hotfix/foobar
         \    0-0----0
          \  /        \
           0—0—--------0
           candidate/1.2.0-SNAPSHOT


    $ git checkout candidate/1.2.0-SNAPSHOT
    $ git merge --no-ff hotfix/foobar
    $ git push origin candidate/1.2.0-SNAPSHOT
    $ git branch -d hotfix/foobar
    $ git push origin :hotfix/foobar

## spin-flow Ideas Consolidated ##

### Concepts ###

- long running branches
  - snapshot
    - typically aka the master branch
    - it only receives accepted features
    - it never receives hot fixes
    - it is never deployed to higher environments
    - it is thus only concerned with major.minor versions
    - candidate branches are taken from it for promotion
- short lived branches
  - feature
    - a requested and engineered change to the functionality of the system
    - it is branched from the current snapshot
    - upon acceptance it will be merged into the current snapshot
  - candidate
    - is branched from the current snapshot at regular intervals to prepare for release
    - allows the master branch to continue rolling forward whilst the release is being prepared
  - hotfix
    - branched from a release tag
    - features from the current snapshot are typically cherry-picked into the hotfix branch
    - applying fixes to the current snapshot first safeguards the system from future regressions
    - it is never merged into snapshot
    - is promoted to a candidate with patch version bump (NB: simplest path is to promote hotfix to higher environments until release without further branching)
- tags
  - release
    - denotes candidates marked for production release
    - no guarantees are made that a release was considered stable in production over time
    - rollbacks to other releases may occur

### Additional Terms ###

- releasing:
  - candidate
  - hotfix
- developing
  - snapshot
  - feature
  - hotfix
- released
  - release tags

### Semantic Versioning ###

#### Background ####

Maven, for example, typically has Major.Minor.Patch-SNAPSHOT to indicate the next release to be Major.Minor.Patch. Our snapshot branch, however, doesn’t and shouldn’t care about patches. Patches are public release concerns. Nor need it care about version increments at all. All of these are opportunity for merge conflicts.

#### Strategies ####

- snapshot and features use “1.x-SNAPSHOT” continually. The release version is yet to be determined.
- candidate use major.minor version bump from latest release tag “2.2.0-SNAPSHOT” via release:branch plugin
- hotfix when branched increments the patch version. e.g., “2.1.0” —> “2.1.1-SNAPSHOT”
- release tags the non-snapshot version via release:prepare release:perform as “2.1.1” for example.

#### Further Reading ####

- https://semver.org
- https://www.cloudbees.com/blog/apache-maven-continuous-deliverydeployment-devoptics-teams-approach


### Handling Rollbacks and Roll-forward Operations ###

The questions that naturally arise from an operational integrity point of view include the following:
- How do we know what the current production release is? This may not necessarily be equivalent to the latest release tag. A rollback may have occurred.
- How do we then know, in the event of a failure to promote a version or a failure discovered some time after its promotion, what the previous version to rollback to is?

#### Possible strategies ####

1. Allocate two additional annotated tags used as aliases: release/current, release/previous and update them as needed. The commit message for the annotated tag could include enough information to indicate what changed.
2. Use a delayed tagging strategy. That is, tag a release and remove the candidate branch after successful deploy. It is still, however, necessary to know what the previous stable release was.
3. Automatically update a file “previous.release” as needed to be used as a pipeline job to “Rollback” if need be. The file will be updated in the snapshot branch and candidate branch if applicable.
4. Use a series of “stable” tags that are applied after some delay post release.

### Database Migrations ###

#### Door 1: separate repo with own pipeline lifecycle ####

Think about it, the DB is a separate micro-service. It services simultaneously multiple versions of the API that connects to it. This occurs
- during the deployment of a new api (two versions are connected simultaneously)
- when a rollback is required (an old version re-connects)
- when a DDL operation occurs (backwards compatibility is required)

The advantage, then, of managing this service for what it is—a separate service—via a separate repository dedicated to this process are as follows:
- pipelines are dedicated to db migrations and relevant tests
- teams are forced by nature to ensure db migrations are backwards compatible

#### Door 2: same repo with additional migration jobs ####

##### Advantages #####

- gives you some level of separation of concerns
- Running all tests regardless

##### Disadvantages #####

- having to deploy the API to multiple environments just to progress the pipeline
- it is unclear if a production release was for db migrations and/or api updates
- the db is not treated as a separate service, even though it is

#### Door 3: same repo, performed by app startup ####

##### Advantages #####

- managed by the app and works everywhere

##### Disadvantages #####

- no separation of concerns
- if migrations take a long time, needs further hand-holding
- it is unclear if a production release was for db and/or api updates

### Default pipeline steps (outline) ###

- feature
  - compile
  - test
  - dev-deploy
  - dev-test
  - accept
    - merge to master
    - delete branch
- snapshot
  - compile
  - test
  - dev-deploy (sonar, containerised deployment)
  - dev-test
  - docs
  - release-prepare
    - update master snapshot version (minor bump)
    - create branch for current snapshot version
- release:branch
  - uat-deploy (apps deploy, code verification)
  - uat-test (app, load)
  - docs
  - confirm
    - release tag
    - delete branch
- release/tag
  - prod-deploy
  - prod-test
  - docs
- hotfix
  - branched from release tag OR release branch
  - compile
  - test
  - dev-deploy (container)
  - accept
    - create release branch (patch bump) OR,
    - merge into current release branch


## Contributions ##

Contributions are more than welcome. If you have suggestions on how to simplify and solidify the process further that'd be fantastic.

Thanks in particular to Adam Ruka (@skinny85) for articulating his oneflow so carefully and providing a helpfully detailed critique of GitFlow.

## Thanks ##

Many thanks to the many who have provided a detailed explanation and discussion on their chosen workflow or commentary on limitations they see in a suggested workflow.
