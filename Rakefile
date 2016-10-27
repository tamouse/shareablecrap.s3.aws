# Just some nice rake tasks to manage the project

require "yaml"
require "erb"


desc "builds thumbnails"
task :thumbs do

end

desc "creates the crap yaml data file from current crap"
task :crapdata do

end

desc "build the site"
task :build => [:thumbs, :crapdata] do
  sh "bundle exec jekyll build"
end

desc "push the site to s3"
task :publish => [:build] do
  sh "s3cmd -rP put #{builddir} #{s3dest}"
end

def builddir
  "_site/"
end

def s3dest
  "s3://shareablecrap/"
end
