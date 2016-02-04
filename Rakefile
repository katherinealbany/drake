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

$lock_file  = "#{$dir}/.lock"
$lock_count = 0

###################################################################################################

task :default => :build

###################################################################################################

task :build, :pull, :cache do |task, args|
  begin
    lock

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

    unlock
  rescue
    unlock
  end
end

###################################################################################################

task :stable, :push do |task, args|
  begin
    lock

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
      raise
    end

    if "#{args[:push]}" == 'true'
      push_image "#{$build_fqdn}"
      push_image "#{$stable_fqdn}"
    end

    if "#{$build_id}" != "#{$stable_id}"
      puts "Image identity mismatch after push!"
      raise
    end

    unlock
  rescue
    unlock
  end
end

###################################################################################################

task :release do
  begin
    lock

    assert_not_empty $dir, 'dir'
    assert_not_empty $repo, 'repo'
    assert_not_empty $release, 'release'

    $release_fqdn = "#{$repo}/#{resolve_image_name $dir}:#{$release}"
    assert_not_empty $release_fqdn, 'release_fqdn'

    if "#{$force}" != 'true'
      sh "docker pull #{$release_fqdn}" do |exists, result|
        if exists
          puts "#{$release_fqdn} already released"
          raise
        end
      end
    end

    Rake::Task['stable'].invoke 'false'

    tag_image $stable_fqdn, $release_fqdn

    $release_id = get_image_id "#{$release_fqdn}"
    assert_not_empty $release_id, 'release_id'

    if "#{$build_id}" != "#{$stable_id}" || "#{$build_id}" != "#{$release_id}"
      puts "Image identity mismatch!"
      raise
    end

    if "#{$push}" == 'true'
      push_image "#{$build_fqdn}"
      push_image "#{$stable_fqdn}"
      push_image "#{$release_fqdn}"
    end

    if "#{$build_id}" != "#{$stable_id}" || "#{$build_id}" != "#{$release_id}"
      puts "Image identity mismatch after push!"
      raise
    end

    unlock
  rescue
    unlock
  end
end

###################################################################################################

def lock
  if File.exist?("#{$lock_file}") and $lock_count == 0
    puts "Already locked!"
    exit 1
  else
    FileUtils.touch($lock_file)
    $lock_count = $lock_count + 1
    puts "Locked: #{$lock_count}"
  end
end

###################################################################################################

def unlock
  if File.exist?("#{$lock_file}") and $lock_count > 0
    $lock_count = $lock_count - 1
    puts "Unlocked: #{$lock_count}"

    if $lock_count == 0
      FileUtils.remove($lock_file)
    end
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
    raise
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
