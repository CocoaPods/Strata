Strata
======

Automate the tasks necessary to develop on the CocoaPods websites

Strata requires having ruby 2.1.3 set up, there are many ways to set up a custom ruby. The CocoaPods development team uses both [rvm](https://rvm.io) and [rbenv](https://github.com/sstephenson/rbenv).

Commands it offers:

```shell
  Strata · (master) ★  ⟩ rake -T
  rake bootstrap[name]      # Runs the Bootstrap task on all the repositories
  rake bootstrap:blog       # Clones the blog.cocoapods.org repository and its dependencies
  rake bootstrap:cocoadocs  # Clones the cocoadocs.org repository and its dependencies
  rake bootstrap:cocoapods  # Clones the cocoapods.org repository and its dependencies
  rake bootstrap:feeds      # Clones the feeds.cocoapods.org repository and its dependencies
  rake bootstrap:humus      # Clones the Humus repository and its dependencies
  rake bootstrap:metrics    # Clones the metrics.cocoapods.org repository and its dependencies
  rake bootstrap:search     # Clones the search.cocoapods.org repository and its dependencies
  rake bootstrap:trunk      # Clones the trunk.cocoapods.org repository and its dependencies 
  rake clone[name]          # Clones the web repositories
  rake db:create            # Create databases for web properties
  rake db:drop              # Drop databases for web properties
  rake db:migrate           # Migrate databases for web properties
  rake db:reset             # Reset databases for web properties
  rake install_system_deps  # Installs application dependencies
  rake issues               # Gets the count of the open issues
  rake pull                 # Pulls all the repositories & updates their submodules
  rake spec                 # Run all specs of all the sites
  rake switch_to_ssh        # Points the origin remote of all the git repos to use the SSH URL
```

### Getting started
```sh
  git clone https://github.com/cocoapods/Strata.git
  cd Strata
  bundle install
  
  # Downloads all the website repos
  rake clone
  
  # Sets all the website repos up
  rake bootstrap:all

  # Downloads apps ( this can set up your database for you )
  rake install_system_deps
```


If the `bootstrap` command fails for a given project/repo, you may need to `cd` into that repo and run `bundle install` followed by `rake bootstrap` inside that directory.

To initialize the database, `cd` into the `Humus` directory within Strata and run `rake db:bootstrap`.


### Serving a project

Once everything is set up, you can `cd` into the repo that you want to make changes to, and run `rake serve` inside that repo. Best to read the README for that project too, they all have their own constraints and tooling.


### Making a Pull Request

Let's say you want to make a change specifically on cocoapods.org. You've already set up your copy of the site, and you've made a change. 

You're going to want to make a fork of [our cocoapods.org repo](https://github.com/cocoapods/cocoapods.org/) and then add that as a new remote to the one inside Strata. e.g `git remote add orta https://github.com/orta/cocoapods.org`. You can then push your commit to your fork with `git push orta master` and create a PR from your fork.