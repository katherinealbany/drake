###################################################################################################

require 'rake'
require 'fileutils'

###################################################################################################

$dir     = ENV['dir']     ||= '.'
$repo    = ENV['repo']    ||= 'katherinealbany'
$build   = ENV['build']   ||= 'latest'
$stable  = ENV['stable']  ||= 'stable'
$release = ENV['release'] ||= 'v1.0.0'
$force   = ENV['force']   ||= 'false'
$push    = ENV['push']    ||= 'true'

###################################################################################################

$lock = '.lock'

###################################################################################################

task :default => :build

###################################################################################################

task :lock do
  puts "#{$dir}/#{$lock}"
  begin
    file = File.stat($lock)
    puts "#{file.pwd}"
    puts 'Already Locked!'
    exit 1
  rescue
    FileUtils.touch($lock)
    puts 'Locked!'
  end
end

# Implement a locking solution
#
#  begin
#    raise "something's wrong here"
#  rescue
#    rollback()
#    raise "error executing task"
#  end

###################################################################################################

task :build, :pull, :cache do |task, args|
  args.with_defaults(:pull => 'false')
  args.with_defaults(:cache => 'true')

  assert_not_empty $dir, 'dir'
  assert_not_empty $repo, 'repo'
  assert_not_empty $build, 'build'

  $build_fqdn = "#{$repo}/#{resolve_image_name $dir}:#{$build}"
  assert_not_empty $build_fqdn, 'build_fqdn'

  sh "docker build --pull=#{args[:pull]} --no-cache=#{to_no_cache args[:cache]} --tag=#{$build_fqdn} #{$dir}"

  $build_id = get_image_id "#{$build_fqdn}"
  assert_not_empty $build_id, 'build_id'
end

###################################################################################################

task :stable, :push do |task, args|
  args.with_defaults(:push => "#{$push}")

  assert_not_empty $dir, 'dir'
  assert_not_empty $repo, 'repo'
  assert_not_empty $stable, 'stable'

  $stable_fqdn = "#{$repo}/#{resolve_image_name $dir}:#{$stable}"
  assert_not_empty $stable_fqdn, 'stable_fqdn'

  Rake::Task['build'].invoke 'true', 'false'

  tag_image $build_fqdn, $stable_fqdn

  $stable_id = get_image_id "#{$stable_fqdn}"
  assert_not_empty $stable_id, 'stable_id'

  if "#{$build_id}" != "#{$stable_id}"
    puts "Image identity mismatch!"
    exit 1
  end

  if "#{args[:push]}" == 'true'
    push_image "#{$build_fqdn}"
    push_image "#{$stable_fqdn}"
  end

  if "#{$build_id}" != "#{$stable_id}"
    puts "Image identity mismatch after push!"
    exit 1
  end
end

###################################################################################################

task :release do
  assert_not_empty $dir, 'dir'
  assert_not_empty $repo, 'repo'
  assert_not_empty $release, 'release'

  $release_fqdn = "#{$repo}/#{resolve_image_name $dir}:#{$release}"
  assert_not_empty $release_fqdn, 'release_fqdn'

  if "#{$force}" != 'true'
    sh "docker pull #{$release_fqdn}" do |exists, result|
      if exists
        puts "#{$release_fqdn} already released"
        exit 1
      end
    end
  end

  Rake::Task['stable'].invoke 'false'

  tag_image $stable_fqdn, $release_fqdn

  $release_id = get_image_id "#{$release_fqdn}"
  assert_not_empty $release_id, 'release_id'

  if "#{$build_id}" != "#{$stable_id}" || "#{$build_id}" != "#{$release_id}"
    puts "Image identity mismatch!"
    exit 1
  end

  if "#{$push}" == 'true'
    push_image "#{$build_fqdn}"
    push_image "#{$stable_fqdn}"
    push_image "#{$release_fqdn}"
  end

  if "#{$build_id}" != "#{$stable_id}" || "#{$build_id}" != "#{$release_id}"
    puts "Image identity mismatch after push!"
    exit 1
  end
end

###################################################################################################

task :get do
  assert_not_empty dir, 'dir'
  assert_not_empty repo, 'repo'
  sh "docker pull --all-tags=true #{repo}/#{resolve_image_name dir}"
  Rake::Task['show'].invoke
end

###################################################################################################

task :show do
  assert_not_empty dir, 'dir'
  assert_not_empty repo, 'repo'
  sh "docker images #{repo}/#{resolve_image_name dir}"
end

###################################################################################################

def get_image_id (image)
  id = `docker inspect --format='{{ .ID }}' #{image}`
  assert_not_empty id, 'id'
  return "#{id}"
end

###################################################################################################

def assert_not_empty (value, name)
  if "#{value}".empty?
    puts "#{name}: must not be null"
    exit 1
  end
end

###################################################################################################

def resolve_image_name (dir)
  pwd = `cd #{dir} && pwd`
  basename = `basename #{pwd}`
  return "#{basename}".gsub("\n", '')
end

###################################################################################################

def to_no_cache (value)
  if "#{value}" == 'true'
    return 'false'
  else
    return 'true'
  end
end

###################################################################################################

def tag_image (src, dest)
  assert_not_empty src, 'src'
  assert_not_empty dest, 'dest'
  sh "docker tag -f #{src} #{dest}"
end

###################################################################################################

def push_image (image)
  assert_not_empty image, 'image'
  sh "docker push #{image}"
end

###################################################################################################

def show_images (image)
  assert_not_empty image, 'image'
  sh "docker images #{image}"
end

###################################################################################################
