# Workflow Documentation

## Managing Workflow Updates

By using prebuilt Docker containers that are managed by the Carpentries core Workbench maintainers, these workflows are designed
to be rarely updated.

However, is important to be able to keep them up-to-date when appropriate.
You can do this locally using your own R and Workbench installation, or via the "04 Maintain: Update Workflow Files"
(`update-workflows.yaml`) GitHub Action.

### Updating locally

In a terminal/git bash, navigate to the lesson folder where you want to update the workflows.

Then, start an R session and:

```r
# Install/Update sandpaper
options(repos = c(carpentries = "https://carpentries.r-universe.dev/",
  CRAN = "https://cloud.r-project.org"))
install.packages("sandpaper")

# update the workflows in your lesson
library("sandpaper")
sandpaper::update_github_workflows()
quit()
```

And then in a bash prompt/git bash terminal:

```bash
git add .github/workflows
git commit -m "Manual update to docker workflows"
git push origin main
```

> [!NOTE]
> For non-renv lessons, this is all the setup you need!
>
> For renv-enabled lessons:
>
> - Cancel any "01 Maintain: Build and Deploy Site" workflow currently running
> - Run the "02 Maintain: Check for Updated Packages" workflow and merge any PR opened to update the renv lockfile
> - This should automatically run the "03 Maintain: Apply Package Cache" workflow to install packages and build the cache
> - A successful cache build should then trigger the "01 Maintain: Build and Deploy Site" workflow

### Updating using GitHub

#### Official lessons

 1. checks out the lesson
 2. provisions the following resources
    - R
    - pandoc
    - lesson infrastructure (stored in a cache)
    - lesson dependencies if needed (stored in a cache)
 3. builds the lesson via `sandpaper:::ci_deploy()`

To update the workflows, either:

- wait for the scheduled run of the "04 Maintain: Update Workflow Files" at approximately midnight every Tuesday
- go to the Actions tab on GitHub, click "04 Maintain: Update Workflow Files" on the left, then "Run Workflow" on the right

This workflow has two caches; one cache is for the lesson infrastructure and
the other is for the lesson dependencies if the lesson contains rendered
content. These caches are invalidated by new versions of the infrastructure and
the `renv.lock` file, respectively. If there is a problem with the cache,
manual invaliation is necessary. You will need maintain access to the repository
and you can either go to the actions tab and [click on the caches button to find
and invalidate the failing cache](https://github.blog/changelog/2022-10-20-manage-caches-in-your-actions-workflows-from-web-interface/)
or by setting the `CACHE_VERSION` secret to the current date (which will
invalidate all of the caches).

#### Your own lessons

This presumes you:

- already have a lesson repository available on GitHub
- have enabled workflows in the lesson repo
- have set up a SANDPAPER_WORKFLOW personal access token (PAT) in the lesson repo

These workflows run on a schedule and at the maintainer's request. Because they
create pull requests that update workflows/require the downstream actions to run,
they need a special repository/organization secret token called
`SANDPAPER_WORKFLOW` and it must have the `public_repo` and `workflow` scope.

Once set up, run the "04 Maintain: Update Workflow Files" (`update-workflows.yaml`) action.

If you want to use your personal account: you can go to
<https://github.com/settings/tokens/new?scopes=public_repo,workflow&description=Sandpaper%20Token>
to create a token. Once you have created your token, you should copy it to your
clipboard and then go to your repository's settings > secrets > actions and
create or edit the `SANDPAPER_WORKFLOW` secret, pasting in the generated token.

If you do not specify your token correctly, the runs will not fail and they will
give you instructions to provide the token for your repository.

## Package Caches for RMarkdown Lessons

The {sandpaper} repository was designed to do as much as possible to separate
the tools from the content. For local builds, this is true, but
there is a minor issue when it comes to workflow files: they must live inside
the repository.

This workflow ensures that the workflow files are up-to-date. The way it work is
to download the update-workflows.sh script from GitHub and run it. The script
will do the following:

### Caching

The two cache management workflows are separated to ensure that once you have a successful build with a working renv cache, this
cache is stored and will be reused by the Workbench Docker container.
This means that lesson builds will be faster once an renv cache is created and reused by the Docker container.

Another major bonus of this setup is that you can keep using this cache indefinitely to build your lesson.
This is important if you need very specific versions of R packages ("pinning").

If and when you want to perform an update to the cache, you can re-run the "02 Maintain: Check for Updated Packages" and verify
that your lesson still builds with the new packages.
If all looks good, re-run the "03 Maintain: Apply Package Cache" workflow, and this will write a new renv cache file to GitHub.

For lessons that have generated content, we use {renv} to ensure that the output
is stable. This is controlled by a single lockfile which documents the packages
needed for the lesson and the version numbers. This workflow is skipped in
lessons that do not have generated content.

Because the lessons need to remain current with the package ecosystem, it's a
good idea to make sure these packages can be updated periodically. The
update cache workflow will do this by checking for updates, applying them in a
branch called `updates/packages` and creating a pull request with _only the
lockfile changed_.

From here, the markdown documents will be rebuilt and you can inspect what has
changed based on how the packages have updated.

## Pull Request and Review Management

Because our lessons execute code, pull requests are a security risk for any
lesson and thus have security measures associated with them. **Do not merge any
pull requests that do not pass checks and do not have bots commented on them.**

This series of workflows all go together and are described in the following
diagram and the below sections:

![Graph representation of a pull request](https://carpentries.github.io/sandpaper/articles/img/pr-flow.dot.svg)

### Pre Flight Pull Request Validation (pr-preflight.yaml)

This workflow runs every time a pull request is created and its purpose is to validate that the pull request is okay to run.
This means the following things:

1. The pull request does not contain modified workflow files
2. If the pull request contains modified workflow files, it does not contain
   modified content files (such as a situation where @carpentries-bot will
   make an automated pull request)
3. The pull request does not contain an invalid commit hash (e.g. from a fork
   that was made before a lesson was transitioned from styles to use the
   workbench).

Once the checks are finished, a comment is issued to the pull request, which
will allow maintainers to determine if it is safe to run the
"Receive Pull Request" workflow from new contributors.

### Receive Pull Request (docker_pr_receive.yaml)

**Note of caution:** This workflow runs arbitrary code by anyone who creates a
pull request. GitHub has safeguarded the token used in this workflow to have no
privileges in the repository, but we have taken precautions to protect against
spoofing.

This workflow is triggered with every push to a pull request.
If this workflow is already running and a new push is sent to the pull request, the workflow running from the previous push will
be cancelled and a new workflow run will be started.

The first step of this workflow is to check if it is valid (e.g. that no
workflow files have been modified). If there are workflow files that have been
modified, a comment is made that indicates that the workflow is not run. If
both a workflow file and lesson content is modified, an error will occur.

The second step (if valid) is to build the generated content from the pull request.
This builds the content and uploads three artifacts:

1. The pull request number (pr)
2. A summary of changes after the rendering process (diff)
3. The rendered files (build)

Because this workflow builds generated content, it follows the same general
process as the `sandpaper-main` workflow with the same caching mechanisms.

The artifacts produced are used by the next workflow.

### Comment on Pull Request (pr-comment.yaml)

This workflow is triggered if the `docker_pr_receive.yaml` workflow is successful.
The steps in this workflow are:

1. Test if the workflow is valid and comment the validity of the workflow to the pull request.
2. If it is valid: create an orphan branch with two commits: the current state of the repository and the proposed changes.
3. If it is valid: update the pull request comment with the summary of changes

Importantly: if the pull request is invalid, the branch is not created so any malicious code is not published.

From here, the maintainer can request changes from the author and eventually
either merge or reject the PR. When this happens, if the PR was valid, the
preview branch needs to be deleted.

### Send Close PR Signal (pr-close-signal.yaml)

Triggered any time a pull request is closed.
This emits an artifact that is the pull request number for the next action.

### Remove Pull Request Branch (pr-post-remove-branch.yaml)

Triggered by `pr-close-signal.yaml`. This removes the temporary branch associated with
the pull request (if it was created).
