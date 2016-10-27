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
  puts "crapdata: #{crapdata(files).inspect} "

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
  File.write(crapdatafile, crapdata(files))
end

desc "build the site"
task :build => [:thumbs, :crapdata] do
  sh "bundle exec jekyll build"
end

desc "push the site to s3"
task :publish => [:build] do
  sh "s3cmd -rP sync #{builddir} #{s3dest}"
end

def assetdir
  @assetdir ||= File.expand_path('../_assets', __FILE__)
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

def baseurl
  aws_config['baseurl']
end

def bucket
  aws_config['bucket']
end

def builddir
  "_site/"
end

def copy_default_thumb(otherfiles, thumbdir)
  otherfiles.each do |file|
    FileUtils.cp default_thumb, thumbfile(file)
  end
end

def crapdata(files)
  files.map do |file|
    {
      "crap" => remove_local_path(file),
      "thumb" => remove_local_path(thumbfile(file)),
      "title" => File.basename(file, '.*')
    }
  end.to_yaml
end

def crapdatafile
  File.join(datadir, 'crap.yml')
end

def crapdir
  File.join(File.expand_path('../crap', __FILE__))
end

def datadir
  @datadir ||= File.expand_path('../_data', __FILE__)
end

def default_thumb
  @default_thumb ||= make_default_thumb
end

def files
  @files ||= Dir[File.join(crapdir, '*')].reject{|file| file.match(%r{\A\.})} # no dot-files
end

def imagefiles
  @imagefiles ||= []
end

def is_image?(file)
  file.match(%r{\.(jpe?g|png|gif|tiff?|svg)$})
end

def local_path
  File.join(File.expand_path('../', __FILE__), '')
end

def make_default_thumb
  sh make_thumbs_cmd([File.join(assetdir, 'default_thumb.png')], assetdir)
  File.join(assetdir, 'default_thumb.gif')
end

def make_thumbs_cmd(imagefiles, thumbdir)
  %Q{mogrify -path #{thumbdir} -format gif -quality 40 -thumbnail #{thumbsize}^ -gravity center -extent #{thumbsize} #{prepare(imagefiles)}}
end

def otherfiles
  @otherfiles ||= []
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

def prepare(words)
  words.map do |word|
    word.shellescape
  end.join(' ')
end

def remove_local_path(file)
  file.sub(local_path, '')
end

def s3dest
  bucket
end

def thumbfile(filename)
  thumbname = [File.basename(filename, '.*'), File.extname(default_thumb)].join('')
  File.join(thumbdir, thumbname)
end

def thumbdir
  @thumbdir ||= File.expand_path('../thumbs', __FILE__)
    .tap{|thumbdir| FileUtils.rm_rf(thumbdir) && FileUtils.mkdir_p(thumbdir)}
end

def thumbsize
  '100x100'
end
