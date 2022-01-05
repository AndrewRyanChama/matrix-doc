# How to release the specification

The whole specification is now released as a single unit/artifact. This document is
the process for releasing the specification and a description of how the (public)
machinery works.

## Prerequisites / preparation

First, can we even release the spec? This stage is mostly preparation work needed
to ensure a consistent and reliable specification.

1. Ensure `main` is committed with all the spec changes you expect to be there.
2. Review the changelog to look for typos, wording inconsistencies, or lines which
   can be merged. For example, "Fix typos" and "Fix spelling" can be condensed to
   "Fix various typos throughout the specification".
3. Do a quick skim to ensure changelogs reference the MSCs which brought the changes
   in. They should be linked to the GitHub MSC PR (not the markdown document).

## The release

Assuming the preparation work is complete, all that remains is the actual specification
release.

1. Create a `release/v1.2` branch where `v1.2` is the version you're about to release.
2. Update the `params.version` section of `config.toml` to use the following template:
   ```toml
   [params.version]
   status = "stable"
   current_version_url = "https://spec.matrix.org/latest"

   # This will be the spec version you're releasing. If that's v1.2, then `major = "1"`
   # and `minor = "2"`
   major = "1"
   minor = "2"

   # Today's date. Please use the format implied here for consistency.
   release_date = "October 01, 2021"
   ```
3. Commit the changes.
4. Tag the branch with the spec release with a format of `v1.2` (if releasing Matrix 1.2).
5. Push the release branch and the tag.
6. GitHub Actions will run its build steps. Wait until these are successful. If fixes
   need to be made to repair the pipeline or spec build, delete and re-tag the release.
7. Check out `main`.
8. Generate the changelog. This is done *after* the tagging to ensure the rendered
   changelog makes sense.
   1. Activate your python virtual environment.
   2. Run `./scripts/generate-changelog.sh v1.2 "October 01, 2021"` (using the correct
      version number and same `release_date` format from the hugo config).
   3. Commit the result.
9. Create a new release on GitHub from the newly created tag.
   * The title should be just "v1.2" (for example).
   * The description should be a copy/paste of the changelog. The generated changelog
     will be at `content/partials/changelogs/v1.2.md` - copy/paste verbatim.
   * Upload the artifacts of the GitHub Actions build for the release to the GitHub
     release as artifacts themselves. This should be the tarball that will be deployed
     to spec.matrix.org.
10. Commit a reversion to `params.version` of `config.toml` on `main`:
    ```toml
    [params.version]
    status = "unstable"
    current_version_url = "https://spec.matrix.org/latest"
    # major = "1"
    # minor = "2"
    # release_date = "October 01, 2021"
    ```
11. Push pending commits and ensure the unstable spec updates accordingly from the
    GitHub Actions pipeline.
12. Deploy the release on the webserver. See internal wiki.

## Patching a release

From time to time we'll need to update a release in the wild. Examples include fixing typos,
updating build machinery, etc. Typically it is not considered a good idea to patch a release
more than 1 month after the original release date - this is because the administrative effort
is typically best reserved for the next release cycle.

**Patch releases are not to be used for spec changes. Only typos and equivalent.**

1. Add the required changes to the release branch (`release/v1.2` for example).
2. Fast forward the `v1.2` tag to the release branch head.
3. Push both the release branch and fast-forwarded tag.
4. Wait for the GitHub Actions build to complete on the tag.
5. Update the assets on the GitHub release to those generated by the latest Actions build.
6. Deploy the release on the webserver. See internal wiki.