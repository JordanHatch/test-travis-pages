# Partly copied from http://evansosenko.com/posts/automatic-publishing-github-pages-travis-ci/

require 'tmpdir'

def destination
  '_site'
end

desc 'Build the Jekyll site'
task :build do
  system "bundle exec jekyll build --destination #{destination}"
end

desc 'Generate deck from Travis CI and publish to GitHub Pages.'
task :travis do
  # if this is a pull request, do a simple build of the site and stop
  if ENV['TRAVIS_PULL_REQUEST'].to_s.to_i > 0
    puts 'Pull request detected. Executing build only.'
    system 'bundle exec rake build'
    next
  end

  repo = ENV['DESTINATION'].gsub(/^git:/, 'https:').strip
  subdir = ENV['DIRECTORY']

  deploy_url = repo.gsub %r{https://}, "https://#{ENV['GH_TOKEN']}@"
  deploy_branch = 'gh-pages'
  rev = %x(git rev-parse HEAD).strip[0..8]

  origin = %x(git config remote.origin.url)
  friendly_origin_name = origin.sub(/^git@github.com:/,'').sub(/.git$/,'')

  Dir.mktmpdir do |dir|
    dir = File.join dir, 'site'
    system 'bundle exec rake build'
    if Dir.exists? destination
      puts "Built site to #{destination}"
    else
      fail "Build failed."
    end
    system "git clone --branch #{deploy_branch} #{repo} #{dir}"
    system "mkdir -p #{dir}/#{subdir}"
    system %Q(rsync -rt --del --exclude=".git" --exclude=".nojekyll" #{destination}/* #{dir}/#{subdir}/)
    Dir.chdir dir do
      puts `git status`

      # setup credentials so Travis CI can push to GitHub
      system "git config user.name '#{ENV['GIT_NAME']}'"
      system "git config user.email '#{ENV['GIT_EMAIL']}'"

      system 'git add --all'
      system "git commit -m 'Built from #{friendly_origin_name}##{rev}'."
      system "git push -q #{deploy_url} #{deploy_branch}"
    end
  end
end
