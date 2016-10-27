# Just some nice rake tasks to manage the project

require "yaml"
require "erb"
require "shellwords"

task :check do
  puts "builddir: #{builddir.inspect}"
  puts "s3dest: #{s3dest.inspect}"
  puts "imagefiles: #{imagefiles.inspect}"
  puts "otherfiles: #{otherfiles.inspect}"
  puts "thumbdir: #{thumbdir.inspect}"
  puts "files: #{files.inspect}"
  puts "crapdir: #{crapdir.inspect}"
  puts "default_thumb: #{default_thumb.inspect}"
  @imagefiles, @otherfiles = partition_files(files)
  puts "iamgefiles: #{imagefiles.inspect} (after)"
  puts "otherfiles: #{otherfiles.inspect} (after)"
  puts "make_thumbs_cmd: #{make_thumbs_cmd(imagefiles, thumbdir).inspect}"
  puts "remove_local_path: #{remove_local_path(imagefiles.first).inspect}"
  puts "local_path: #{local_path.inspect}"
  puts "datadir: #{datadir.inspect}"
  puts "aws_config: #{aws_config.inspect}"
  puts "baseurl: #{baseurl.inspect}"
  puts "bucket: #{bucket.inspect}"

end

task :partition_files do
  @imagefiles, @otherfiles = partition_files(files)
end

desc "builds thumbnails"
task :thumbs => :partition_files do
  sh make_thumbs_cmd(imagefiles, thumbdir)
  copy_default_thumb(otherfiles, thumbdir)
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

def is_image?(file)
  file.match(%r{\.(jpe?g|png|gif|tiff?|svg)$})
end

def make_thumbs_cmd(imagefiles, thumbdir)
  %Q{mogrify -path #{thumbdir} -format gif -quality 40 -thumbnail 150x150^ -gravity center -extent 150x150 #{prepare(imagefiles)}}
end

def copy_default_thumb(otherfiles, thumbdir)
  otherfiles.each do |file|
    FileUtils.cp default_thumb, File.join(thumbdir, thumbfile(File.basename(file)))
  end
end

def thumbfile(filename)
  [File.basename(filename, '.*'), File.extname(default_thumb)].join('.')
end


def imagefiles
  @imagefiles ||= []
end

def otherfiles
  @otherfiles ||= []
end

def prepare(words)
  words.map do |word|
    word.shellescape
  end.join(' ')
end


def thumbdir
  @thumbdir ||= File.expand_path('../thumbs', __FILE__)
    .tap{|thumbdir| FileUtils.rm_rf(thumbdir) && FileUtils.mkdir_p(thumbdir)}
end

def files
  @files ||= Dir[File.join(crapdir, '*')].reject{|file| file.match(%r{\A\.})} # no dot-files
end

def crapdir
  File.join(File.expand_path('../crap', __FILE__))
end

def partition_files(files)
  images = []
  nonimages = []
  files.each do |file|
    if is_image?(file)
      images << file
    else
      nonimages << file
    end
  end
  [images, nonimages]
end

def default_thumb
  @default_thumb ||= File.expand_path('../_assets/default_thumb.gif', __FILE__)
end


def remove_local_path(file)
  file.sub(local_path, '')
end

def local_path
  File.join(File.expand_path('../', __FILE__), '')
end

def aws_config
  @aws_config ||= YAML.load(
    ERB.new(
      File.read(
        File.join(datadir, 'aws.yml')
        )
      ).result
    )
end

def datadir
  @datadir ||= File.expand_path('../_data', __FILE__)
end

def bucket
  aws_config['bucket']
end

def baseurl
  aws_config['baseurl']
end
