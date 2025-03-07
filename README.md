# wiki-actions-poc

This serves as a proof of concept of how BTA would use Github Actions to automatically generate wiki pages

# How it works

Now the fun begins.

As an Org, BTA has 2,000 Github Actions minutes per month. By default, the Org should have a spending limit of $0, which means no [further minutes would be able to be used](https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions#about-spending-limits). But 2,000 minutes should be far in excess of what BTA need, assuming tactical usage. Long-running executions, like the importer, are not a viable use-case, as without a serious optimization pass, that can run hours. Ergo, python generation and REST API posts only.

These Actions would run on various triggers, usually PR creation or merges to a branch.

That's still not great for run time, as during hotfix-intense period the Org could really run up that number. To further restrict how frequently this runs, BTA has a few options:

1. Approvals. Unfortunately, in private repos this is restricted to paid acounts because haha Microsoft billing practices. The repo could be made public to get around this.
2. Creative branch targeting. Rather than trigger on merges to `master`, it could trigger on merge to `wiki`. This takes a bit out of the automation, but it does make it much more intentional.
3. Approvals on a separate repo like the wiki gen repo. Rather than checkout the wiki gen repo, the wiki gen repo would checkout the BTA repo and run there. I'm not clear on what needs to be public/private and why, but the *contents* of any given files shouldn't be exposed this way. 

Of those 3 (I can probably come up with more), #2 feels like the sanest option and this iteration will proceed with that in mind. 

Posting privileges to the wiki is managed by Github Repository Secrets (accessed via Settings). These get injected into the Actions runner during runtime. Access to those Secrets can be fairly granularly managed via Organization teams, if that is a concern. 

## Iteration 1

The code is present in [`wiki-merge.yml`](.github/workflows/wiki-merge.yml) and is publicly documented in GitHub's [docs](https://docs.github.com/en/actions). Good luck.

1. On merge to `wiki`, the Github Action spins up.
2. It checks out the the ~~repo~~ archived BTA repo and external code (currently the wiki-gen-poc). In the actual BTAU repo, it would check itself out.
3. It runs the appropriate script(s) against the archived repo because I realized I needed access to the whole-ass codebase. On an error, the Action exits immediately and cleans itself up. On a success, job's done. This is eminently configurable. 

View my mess of trying to figure this out [here](https://github.com/amidatelion/wiki-actions-poc/actions) and ask any questions that come up. 

Currently runs at 2m 17s for just Faction Stores. An async http implementation will probably lop off a good 25-50% of that, but that's more research for me. 

[Final Result](https://www.bta3062.com/index.php?title=WikiGenPoc), with broken links left in for "good enough" theory's sake (i.e. will be manually replaced by redirects or will use [jinja conditionals](https://github.com/amidatelion/wiki-gen-poc/tree/parser?tab=readme-ov-file#next-steps) to get to *mostly* the right place)

# Next Steps

1. Verify assumptions re: Run Restrictions with BD. i.e BD please ask questions and select from...
    1. Approvals in BTAU repo. Requires making it public.
    2. Target `wiki` branch. Less automation, more intentional usage. Current assumption, happy to change/discuss.
    3. Approvals in the wiki automation repo. 
2. Implement the POC in the BTA Org space. There are gotchas with private repos I'd want to examine. 
3. Investigate running specific scripts only if commits run against specific folders (i.e. don't run everything if only faction shops or factory worlds get updated)