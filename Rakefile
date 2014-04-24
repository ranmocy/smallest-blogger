begin
  require 'pathname'
  require 'fileutils'
  load 'Guardfile'
rescue
  nil
end

# HOST           = "http://ranmocy.info"
GITHUB_REPO    = "git@github.com:ranmocy/ranmocy.github.io.git"
GITHUB_BRANCH  = "master"
GITCAFE_REPO   = "git@gitcafe.com:ranmocy/ranmocy.git"
GITCAFE_BRANCH = "gitcafe-pages"
SILENT         = ($VERBOSE) ? "" : ">/dev/null 2>/dev/null"


desc "Generate blog files"
task :generate do
  Scanner.new.call(self, :start_begin)
  Generator.new.call(self, :start_begin)
end

desc "Update sources"
task :upload, [:force] do |t, args|
  system("git push github master:source #{args[:force] ? '-f' : ''} #{SILENT}") ? puts("Sourced to Github.") : puts("Failed sourcing to Github.")
  system("git push gitcafe master:master #{args[:force] ? '-f' : ''} #{SILENT}") ? puts("Sourced to GitCafe.") : puts("Failed sourcing to GitCafe.")
end

desc "Generate and publish blog to Github and GitCafe"
task :publish => [:generate, :upload] do
  Dir.chdir BUILD_PATH do
    system("git init #{SILENT}")
    system("git add --all #{SILENT}")
    message = "Site updated at #{Time.now.utc}"
    system("git commit -m #{message.shellescape} #{SILENT}") && puts("Commited.")

    system("git push #{GITHUB_REPO} master:#{GITHUB_BRANCH} --force #{SILENT}") ? puts("Success published Github") : warn("Failed published Github")
    system("git push #{GITCAFE_REPO} master:#{GITCAFE_BRANCH} --force #{SILENT}") ? puts("Success published GitCafe") : warn("Failed published GitCafe")
  end
end
