# wiki-actions-poc

This serves as a proof of concept of how BTA would use Github Actions to automatically generate wiki pages

# How it works
As an Org, BTA has 2,000 Github Actions minutes per month. By default, the Org should have a spending limit of $0, which means no [further minutes would be able to be used](https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions#about-spending-limits). But 2,000 minutes should be far in excess of what BTA need, assuming tactical usage. Long-running executions, like the importer, are not a viable use-case, as without a serious optimization pass, that can run hours. Ergo, async python generation and posts only.

These Actions would run on various triggers, usually PR creation or merges to a branch.

That's still not great for run time, as during hotfix-intense period the Org could really run up that number. To further restrict how frequently this runs, BTA has a few options:

1. Approvals. Unfortunately, in private repos this is restricted to paid acounts because haha Microsoft billing practices. The repo could be made public to get around this.
2. Creative branch targeting. Rather than trigger on merges to `master`, it could trigger on merge to `wiki`. This takes a bit out of the automation, but it does make it much more intentional.
3. Approvals on a separate repo like the wiki gen repo. Rather than checkout the wiki gen repo, the wiki gen repo would checkout the BTA repo and run there. I'm not clear on what needs to be public/private and why, but the *contents* of any given files shouldn't be exposed this way. 

Of those 3 (I can probably come up with more), #2 feels like the sanest option and this iteration will proceed with that in mind. 

Access to the wiki is managed by Github Repository Secrets (accessed via Settings). These get injected into the Actions runner during runtime. Access to those Secrets can be fairly granularly managed via Organization teams, if that is a concern. 

## Iteration 1

1. On merge to `wiki`, the Github Action spins up.
2. It checks out the the repo and external code (currently the wiki-gen-poc). 
3. It runs the appropriate scripts. On an error, the Action exits immediately and cleans itself up. On a success, job's done. This is eminently configurable. 

## Next Steps

1. Verify assumptions re: Run Plan with BD.
2. Implement the POC in the BTA Org space. There are gotchas with private repos I'd want to examine. 
3. Investigate running specific scripts only if commits run against specific folders (i.e. don't run everything if only faction shops or factory worlds get updated)







