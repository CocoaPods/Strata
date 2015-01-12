# encoding: utf-8

# @return [Hash<String>] The list of the names of the CocoaPods repositories
#         which are web based and their dependencies.
#

WEB_REPOS = {
  'blog.cocoapods.org' => [],
  'cocoadocs-api' => ['Humus'],
  'cocoadocs.org' => [],
  'cocoapods.org' => ['Humus'],
  'feeds.cocoapods.org' => [],
  'guides.cocoapods.org' => [],
  'Humus' => [],
  'metrics.cocoapods.org' => ['Humus'],
  'search.cocoapods.org' => ['Humus'],
  'trunk.cocoapods.org' => ['Humus'],
  'trunk.cocoapods.org-api-doc' => [],
}

# @return [Array<String>] The list of the repos which should be cloned by
#         default.
#

DEFAULT_REPOS = WEB_REPOS

# Task bootstrap / set-up
#-----------------------------------------------------------------------------#

def setup name
  clone name
  bootstrap name
end

def bootstrap name = nil
  strap = ->(dir) do
    Dir.chdir(dir) do
      subtitle "Bootstrapping #{dir}"
      if has_rake_task?('bootstrap')
        sh "rake --no-search bootstrap"
      end
    end
  end
  
  if name
    title "Bootstrapping the #{name} repository"
    strap.call(name)
  else
    title "Bootstrapping all the repositories"
    rakefile_repos.each do |dir|
      strap.call(dir)
    end
  end

  disk_usage = `du -h -c -d 0`.split(' ').first
  puts "\nDisk usage: #{disk_usage}"
end

def clone name
  repos = fetch_default_repos
  if name
    repos = [repos.find { |repo| repo['name'] == name }]
  end
  
  if repos
    title "Cloning the website repositories"
    clone_repos(repos)
  else 
    title "Could not find the repo you were looking for"
  end
end

namespace :bootstrap do
  WEB_REPOS.keys.each do |name|
    short_name = name.split('.').first.downcase
    
    desc "Clones the #{name} repository and its dependencies"
    task short_name do
      if system('which bundle')
        WEB_REPOS[name].each do |dependency_name|
          setup dependency_name
        end
        setup name
      else
        $stderr.puts "\033[0;31m" \
          "[!] Please install the bundler gem manually:\n" \
          "    $ [sudo] gem install bundler" \
          "\e[0m"
        exit 1
      end
    end
  end
end


begin

  # Task clone
  #-----------------------------------------------------------------------------#

  desc "Clones the web repositories"
  task :clone, :name do |task, args|
    clone(args.name)
  end
  
  # Task install_system_deps
  #-----------------------------------------------------------------------------#
  desc "Installs application dependencies"
  task :install_system_deps do
    title "Installing application dependencies"
    rakefile_repos.each do |dir|
      Dir.chdir(dir) do
        if has_rake_task?('install_tools')
          subtitle "Installing dependencies of #{dir}"
          sh "rake --no-search install_tools"
        end
      end
    end

    disk_usage = `du -h -c -d 0`.split(' ').first
    puts "\nDisk usage: #{disk_usage}"
  end
  
  # Task bootstrap_repos
  #-----------------------------------------------------------------------------#

  desc "Runs the Bootstrap task on all the repositories"
  task :bootstrap, :name do |task, args|
    bootstrap(args.name)
  end


  # Task switch_to_ssh
  #-----------------------------------------------------------------------------#

  desc "Points the origin remote of all the git repos to use the SSH URL"
  task :switch_to_ssh do
    repos = fetch_default_repos
    title "Setting SSH URLs"
    repos.each do |repo|
      name = repo['name']
      url = repo['ssh_url']
      subtitle(name)
      Dir.chdir(name) do
        sh "git remote set-url origin '#{url}'"
      end
    end
  end

  # Task pull
  #-----------------------------------------------------------------------------#

  desc "Pulls all the repositories & updates their submodules"
  task :pull do
    title "Pulling all the repositories"
    if pull_current_repo(false)
       puts yellow("\n[!] The Strata repository itself has been updated.\n" \
            "You should run `rake bootstrap` to update all repositories\n" \
            'and fetch the potentially new ones.')
    else
      updated_repos = []
      repos.each do |dir|
        Dir.chdir(dir) do
          updated = pull_current_repo(true)
          updated_repos << dir if updated
        end
      end

      unless updated_repos.empty?
        title "Summary"
        updated_repos.each do |dir|
          subtitle dir
          Dir.chdir(dir) do
            puts `git log ORIG_HEAD..`
          end
        end
      end
    end
  end

  desc "Gets the count of the open issues"
  task :issues do
    require 'open-uri'
    require 'json'

    title 'Fetching open issues'
    WEB_REPOS.keys.dup.push('Strata').each do |name|
      url = "https://api.github.com/repos/CocoaPods/#{name}/issues?state=open&per_page=100"
      response = open(url).read
      issues = JSON.parse(response)

      pure_issues = issues.reject { |issue| issue.has_key?('pull_request') }
      pull_requests = issues.select { |issue| issue.has_key?('pull_request') }
      puts cyan("\n#{name}")

      if issues.empty?
        puts green "Awesome no open issues"
      else
        unless pull_requests.empty?
          if pull_requests.count == 1
            puts yellow("1 pull request")
          elsif pull_requests.count > 1
            puts yellow("#{pull_requests.count} pull requests")
          end

          if pull_requests.count <= 5
            puts pull_requests.map{ |i| "- " + i['title'] }
          end
        end

        if pure_issues.count == 100
          puts yellow("100 or more open issues")
        elsif pure_issues.count == 1
          puts yellow("1 open issue")
        elsif pure_issues.count > 1
          puts yellow("#{pure_issues.count} open issues")
        end

        if pure_issues.count <= 5
          puts pure_issues.map{ |i| "- " + i['title'] }
        end
      end
    end
  end

  #-- Spec -------------------------------------------------------------------#

  desc "Run all specs of all the sites"
  task :spec do
    title "Running specs"
    WEB_REPOS.keys.reverse.each do |repo|
      if Dir.exists?(repo)
        Dir.chdir(repo) do
          subtitle repo
          sh 'bundle exec rake spec'
        end
      else
        puts "Skipping running the spec of #{repo} as it is not cloned."
      end
    end
  end

  namespace :db do

    desc "Create databases for web properties"
    task :create do
      title "Creating databases"
      run_humus_rake_task("db:create")
    end

    desc "Drop databases for web properties"
    task :drop do
      title "Dropping databases"
      run_humus_rake_task("db:drop")
    end

    desc "Reset databases for web properties"
    task :reset do
      title "Resetting reset"
      run_humus_rake_task("db:reset")
    end

    desc "Migrate databases for web properties"
    task :migrate do
      title "Performing migration"
      run_humus_rake_task("db:migrate")
    end

  end

rescue LoadError
  $stderr.puts "\033[0;31m" \
    '[!] Some Rake tasks haven been disabled because the environment' \
    ' couldn’t be loaded. Be sure to run `rake bootstrap` first.' \
    "\e[0m"
end

#-----------------------------------------------------------------------------#
# HELPERS
#-----------------------------------------------------------------------------#

# Repos Helpers
#-----------------------------------------------------------------------------#

# @return [Array<Hash>] The list of the CocoaPods repositories which contain a
# Gem as returned by the GitHub API.
#
def fetch_default_repos
  fetch_repos.select do |repo|
    DEFAULT_REPOS.include?(repo['name'])
  end
end

# @return [Array<Hash>] The list of the CocoaPods repositories as returned by
# the GitHub API.
#
def fetch_repos
  require 'json'
  require 'open-uri'
  title "Fetching repositories list"
  url = 'https://api.github.com/orgs/CocoaPods/repos?type=public'
  repos = []
  loop do
    file = open(url)
    response = file.read
    repos.concat(JSON.parse(response))

    link = Array(file.meta['link']).first
    if match = link.match(/<(.*)>; rel="next"/)
      url = match.captures.first
    else
      break
    end
  end

  repos.reject! { |repo| repo['name'] == 'Strata' }
  puts "Found #{repos.count} public repositories"
  repos
end

# Clones the given repos to a directory named after themselves unless the
# directory already exists.
#
# @param  [Array<Hash>] The description of the repositories.
#
# @return [void]
#
def clone_repos(repos)
  repos.each do |repo|
    name = repo['name']
    subtitle "Cloning #{name}"
    url = repo['clone_url']
    if File.exist?(name)
      puts "Already cloned"
    else
      sh "git clone #{url} #{name}"

      # Run the new repo's bundle install
      Dir.chdir(name) do
        Bundler.with_clean_env do 
          system 'bundle install'
        end
      end
    end
  end
end

# Pull the repo in the current working directory
#
# @param [Bool] Whether we want to update the submodules as well
#
# @return [Bool] true if the repo has updates that were pulled
#                false if there was nothing to update (repo already up-to-date or ahead)
#
def pull_current_repo(update_submodules)
  subtitle "Pulling #{File.basename(Dir.getwd)}"
  sh "git remote update"
  status = `git status -uno`
  unless status.include?('up-to-date') || status.include?('ahead')
    sh "git pull --no-commit"
    sh "git submodule update" if update_submodules
    return true
  end
  false
end

# Checks the given repo for a release and fails the task if any issue exits
# listing them.
#
# @param [String] gem_dir The repo to check.
# @param [String] version The version which should be released.
#
def check_repo_for_release(repo_dir, version)
  errors = []
  Dir.chdir(repo_dir) do
    if `git symbolic-ref HEAD 2>/dev/null`.strip.split('/').last != 'master'
      errors << "You need to be on the `master` branch in order to do a release."
    end

    if `git tag`.strip.split("\n").include?(version.to_s)
      errors << "A tag for version `#{version}` already exists."
    end

    diff_lines = `git diff --name-only`.strip.split("\n")

    if diff_lines.size == 0
      errors << "Change the version number of the gem yourself"
    end

    diff_lines.delete('Gemfile.lock')
    diff_lines.delete('CHANGELOG.md')
    unless diff_lines.count == 1
      # TODO Check that is only the version file changed
      error = "Only change the version, the CHANGELOG.md and the Gemfile.lock files"
      error << "\n- " + diff_lines.join("\n- ")
      errors << error
    end

    unless Pathname.new('CHANGELOG.md').read.lines.include?("## #{version}\n")
      errors << "The CHANGELOG.md doesn't include the released version " \
        "`## #{version}`.Update it manually."
    end
  end

  unless errors.empty?
    errors.each do |error|
      $stderr.puts(red("[!] #{error}"))
    end
    exit 1
  end
end

# @return [Array<String>] All the checked out repos
#
def repos
  Dir['*/'].map { |dir| dir[0...-1] }
end

# @return [Array<String>] All the directories that contains a Rakefile,
#
def rakefile_repos
  Dir['*/Rakefile'].map { |file| File.dirname(file) }
end

# @return [Array<String>]
#
def git_branch_list(arguments = nil)
  branches = `git branch #{arguments}`.split("\n")
  branches.map { |line| line.split(' ').last }
end

def default_branch
  default_branches = ['master', 'develop']
  branches = git_branch_list
  common = branches & default_branches
  if common.count == 1
    common.first
  end
end

def make_github_release(repo, version, tag, access_token)
  body = changelog_for_repo(repo, version)

  REST.post("https://api.github.com/repos/CocoaPods/#{repo}/releases",
    {
      tag_name: tag,
      name: version.to_s,
      body: body,
      prerelease: version.prerelease?,
    }.to_json,
    {
      'Content-Type' => 'application/json',
      'User-Agent' => 'runscope/0.1,segiddins',
      'Accept' => '*/*',
      'Accept-Encoding' => 'gzip, deflate',
      'Authorizaton' => "token #{access_token}",
    },
  )
end

def changelog_for_repo(repo, version)
  changelog_path = File.expand_path(repo + '/CHANGELOG.md')
  if File.exists?(changelog_path)
    title_token = '## '
    current_verison_title = title_token +  version.to_s
    text = File.open(changelog_path, "r:UTF-8") { |f| f.read }
    lines = text.split("\n")

    current_version_index = lines.find_index { |line| line =~ (/^#{current_verison_title}/) }
    unless current_version_index
      raise "Update the changelog for the last version (#{version})"
    end
    current_version_index += 1
    previous_version_lines = lines[(current_version_index+1)...-1]
    previous_version_index = current_version_index + (previous_version_lines.find_index { |line| line =~ (/^#{title_token}/) && !line.include?('rc') } || lines.count)

    relevant = lines[current_version_index..previous_version_index]

    relevant.join("\n").strip
  end
end

def run_humus_rake_task(task)
  if Dir.exists?("Humus")
    Dir.chdir("Humus") do
      sh 'rake ' + task + ' RACK_ENV=test'
      sh 'rake ' + task + ' RACK_ENV=development'
    end
  end
end

# Gem Helpers
#-----------------------------------------------------------------------------#

# @return [Array<String>] the directory of the gems.
#
def gem_dirs
  gemspecs = Dir['*/*.gemspec']
  gemspecs.map { |path| File.dirname(path) }.uniq
end

def spec(gem_dir)
  files = Dir.glob("#{gem_dir}/*.gemspec")
  unless files.count == 1
    error("Unable to select a gemspec in #{gem_dir}")
  end
  spec_path = files.first
  spec = Gem::Specification::load(spec_path.to_s)
end

def gem_version(gem_dir)
  spec(gem_dir).version
end

def gem_name(gem_dir)
  spec(gem_dir).name
end

def last_tag(dir)
  Dir.chdir(dir) do
    `git describe --abbrev=0 2>/dev/null`.chomp
  end
end

# Other Helpers
#-----------------------------------------------------------------------------#

# @return [Bool] Whether the Rakefile in the current working directory has a
#         task with the given name.
#
def has_rake_task?(task)
  `rake --no-search --tasks #{task}`.include?("rake #{task}")
end

def silent_sh(command)
  require 'english'
  output = `#{command} 2>&1`
  unless $CHILD_STATUS.success?
    puts output
    exit 1
  end
  output
end

# UI
#-----------------------------------------------------------------------------#

# Prints a title.
#
def title(string)
  puts
  puts "-" * 80
  puts cyan(string)
  puts "-" * 80
end

def subtitle(string)
  puts "\n#{green(string)}"
end

def error(string)
  raise red("[!] #{string}")
end

# Colorizes a string to green.
#
def green(string)
  "\033[0;32m#{string}\e[0m"
end

# Colorizes a string to yellow.
#
def yellow(string)
  "\033[0;33m#{string}\e[0m"
end

# Colorizes a string to red.
#
def red(string)
  "\033[0;31m#{string}\e[0m"
end

# Colorizes a string to cyan.
#
def cyan(string)
  "\033[0;36m#{string}\033[0m"
end
