Strata
======

Automate the tasks necessary to develop on the CocoaPods websites

```shell
  Strata · (master) ★  ⟩ rake -T
  rake bootstrap:blog         # Clones the blog.cocoapods.org repository and its dependencies
  rake bootstrap:cocoadocs    # Clones the cocoadocs.org repository and its dependencies
  rake bootstrap:cocoapods    # Clones the cocoapods.org repository and its dependencies
  rake bootstrap:feeds        # Clones the feeds.cocoapods.org repository and its dependencies
  rake bootstrap:humus        # Clones the Humus repository and its dependencies
  rake bootstrap:metrics      # Clones the metrics.cocoapods.org repository and its dependencies
  rake bootstrap:search       # Clones the search.cocoapods.org repository and its dependencies
  rake bootstrap:trunk        # Clones the trunk.cocoapods.org repository and its dependencies / Clones the trunk.cocoapods.org-api-doc repository and its dependencies
  rake bootstrap_repos[name]  # Runs the Bootstrap task on all the repositories
  rake clone[name]            # Clones the web repositories
  rake issues                 # Gets the count of the open issues
  rake pull                   # Pulls all the repositories & updates their submodules
  rake spec                   # Run all specs of all the sites
  rake switch_to_ssh          # Points the origin remote of all the git repos to use the SSH URL
````