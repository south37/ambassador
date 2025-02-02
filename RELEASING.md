Releasing Ambassador
====================

If you just want to **use** Ambassador, check out https://www.getambassador.io/! You don't need to build - much less release! - anything, and in fact you shouldn't.

If you don't work at Datawire, this document is probably not going to help you. Maybe check out [the developer guide](BUILDING.md) instead.

----

If you're still reading, you must be at Datawire. Congrats, you picked a fine place to work! To release Ambassador, you'll need credentials for our Github repos.

1. PRs will pile up on `master`. **Don't accept PRs for which CI doesn't show passing tests.**

2. Once `master` has all the release drivers, tag `master` with an RC tag, e.g. `v0.77.0-rc1`. **This version tag must start with a 'v'.**

3. The RC tag will trigger CI to run a new build and new tests. It had better pass: if not, figure out why.

4. The RC build will be available as e.g. `quay.io/datawire/ambassador:0.77.0-rc1` and also as e.g. `quay.io/datawire/ambassador:0.77.0-rc-latest`. Any other testing you want to do against this image, rock on.

5. When you're happy with everything, sync up the docs!
   - `make pull-docs` to pull updates from the docs repo
   - Handle conflicts as needed.
   - Commit any conflict-resolution changes to `master`.

6. After the docs are synced, use `make release-prep` to update `CHANGELOG.md` and `docs/versions.yml`.
   - It will _commit_, but not _push_, the updated files. Make sure the diffs it shows you look correct!
   - Do a manual `git push` to update the world with your new files.
   - It is *critical* to update `docs/versions.yml` so that everyone gets the new version.

7. Now for the time-critical bit.
   - Tag `master` with a GA tag like `v0.77.0` and let CI do its thing. **This version tag must start with a 'v'.**
   - CI will retag the latest RC image as the GA image.

8. _After the CI run finishes_:
   - `make push-docs` _after the retag_ to push new docs out to the website.
   - Go to https://github.com/datawire/ambassador/releases and create a new release.
      - `make release-prep` should've output the text to paste into the release description.

   **Note** that there must be at least one RC build before a GA, since the GA tag **does not** rebuild the docker images -- it retags the ones built by the RC build. This is intentional, to allow for testing to happen on the actual artifacts that will be released.

   **Note also** that even though the version _tag_ starts with a 'v', the version _number_ displayed by the diag UI will _not_ start with a 'v'.**

9. Submit a PR into the `helm/charts` repo to update things in `stable/ambassador`. You may be able to ask Support to do this, but if not:
   - in `Chart.yaml`, update `appVersion` with the new Ambassador version, and bump `version`.
   - in `README.md`, update the default value of `image.tag`.
   - in `values.yaml`, update `tag`.
   - Helpful stuff for this:
      - git checkout master               # switch to master
      - git fetch --all --prune           # make sure our view of remotes is up to date
      - git pull                          # pull down any changes to master
      - git rebase upstream/master        # move master on top of upstream
      - git push                          # push rebases to our fork
      - git checkout -b update/$VERSION   # switch to a feature branch
      - make your edits
      - git commit -a                     # commit changes -- don't forget DCO in the message!
      - git push origin update/$VERSION   # push to feature branch
      - open a PR
    - Once your PR is merged, _delete the feature branch without merging it back to origin/master_.

10. Additional updates, which you may be able to ask Marketing to do:
   - Submit a PR to https://github.com/datawire/pro-ref-arch that updates the `env.sh.example` versions.
   - Submit a PR to the Ambassador website repository to update the version on the homepage.
