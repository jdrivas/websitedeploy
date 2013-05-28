require 'highline/import'
require 'json'
require 'aws-sdk'
require 'fileutils'

# Configuration paramaters
# TODO: Should move this into a config file.
source_repo = "git@github.com:jdrivas/websitedeploy.git"
source_directory = "site"
website_staging_bucket = "staging.website.stage3systems.net"
website_production_bucket = "stage3systems.com"

# Internal configuration
aws_key_file_name = ".aws_credentials"

#
# Key Management
#

# save the two AWS keys as JSON in the key_file
def save_keys(file_name, key_id, secret_key)
  File.open(file_name, "w") do |f|
    f.write({
      "aws_access_key_id" => key_id,
      "aws_secret_key" => secret_key
      }.to_json)
  end
  Pathname.new(file_name).chmod(0600)
end

# Read the keys back and return them in a hash.
def get_keys(file_name)
  keys_json = File.open(file_name, "r").read
  parsed = JSON.parse(keys_json)
end

#
# File list
#

# Create a file list of files that will be loaded up to S3 as a website.
# Take a directory and recurse down to get the list.
def get_files(directory_name)
  files = []
  Dir.foreach(directory_name) do |file|
    next if file.match(/^\.{1,2}$/) # ignore '.', '..'
    path = directory_name + "/" + file
    path.gsub!(/^\.\//,"") # remove the leading ./
    if File.directory?(path)
      files += get_files(path)
    else
      files.push(path)
    end
  end
  files
end

# Assume the given directory will be the root of the 
# returned list of files.
def get_file_list(directory_name)
  files = []
  Dir.chdir(directory_name) do
    files = get_files(".")
  end
end

desc "Show the list of files that will be deployed to S3"
task :show_file_list do
  puts "files collected from: #{source_directory}"
  files = get_file_list(source_directory)
  files.each {|f| puts f}
end

#
# S3 Update
#

# TODO: investigate updates for Metadata.
# copy the files up to the bucket.
def update_files(aws_creds, bucket_name, root_directory, file_list, destination_folder=nil)
  # connect
  puts "connecting to AWS."
  s3 = AWS::S3.new(access_key_id: aws_creds["aws_access_key_id"], secret_access_key: aws_creds["aws_secret_key"])
  bucket = s3.buckets[bucket_name]
  if bucket.exists?
    puts "Updating to: #{bucket.name}"
    file_list.each do |f|
      key = destination_folder.nil? ? f : root_directory.gsub("^\.//", "") + "/" + file
      path = Pathname.new(root_directory + "/" + f)
      puts "Updating: #{key}"
      # puts "File: #{path.expand_path}"
      # puts "-"
      bucket.objects.create(key, path.expand_path.read)
    end
  else
    puts "Bucket: \"#{bucket.name}\" doesn't exist on S3."
  end
end

#
# Git clone
#

# Clones a branch. If clone_name is a directory will name the clone the same as the source.
def git_clone_branch(branch, source_repo, clone_name)
  FileUtils.rm_rf clone_name if Pathname.new(clone_name).directory?
  puts "Clonning #{source_repo} to #{clone_name}"
  system "git clone -b #{branch} #{source_repo} #{clone_name}"
end

desc 'Input and save AWS credentials, call this if you want to change your creds.'
task :add_aws_credentials do
  puts "Couldn't find your AWS keys stored locally. I'll do that now."
  puts "Your keys will be stored in the file: #{aws_key_file_name}."
  puts "The file should be in your .gitignore as well. Don't change this."
  puts "YOU DO NOT WANT TO STORE YOUR KEYS IN GIT"
  aws_access_key_id = ask("AWS ACCESS KEY ID: ")
  aws_secret_key = ask("AWS Secret Key: ")
  puts("Here is what I got")
  puts "AWS ACCESS KEY ID: \"#{aws_access_key_id}\""
  puts "AWS Secret Key: \"#{aws_secret_key}\""
  puts "These will be stored in #{aws_key_file_name} in json format."
  overwrite_keys = File.exist?(aws_key_file_name) && agree("#{aws_key_file_name} exists. Overwrite? ")
  save_keys(aws_key_file_name, aws_access_key_id, aws_secret_key) if overwrite_keys || !File.exist?(aws_key_file_name)
end

# Executes the task to create the key file if it if missing.
file aws_key_file_name do
  Rake::Task[:add_aws_credentials].execute
end

# We could use rake arguments but the syntax is a bit wierd and 
# cumbersome in some shells (e.g. zsh). So we'll just 
# create tasks with the deploy environment embedded (staging, production)
desc 'Deploy to staging' 
task :deploy_staging => aws_key_file_name do

  # do a git clone of the staging branch to /tmp
  tmp_clone = "/tmp/staging_clone"
  website_source = tmp_clone + "/" + source_directory
  git_clone_branch(:staging, source_repo, tmp_clone)

  # Make a list of the files to upload
  files = get_file_list(website_source)

  # upload the files.
  keys = get_keys(aws_key_file_name)
  update_files(keys, website_staging_bucket, website_source, files)

  # remove the tmp clone
  puts "Removing: #{tmp_clone}"
  FileUtils.rm_rf tmp_clone

end

desc 'Deploy to production'
task :deploy_production => aws_key_file_name do

  tmp_clone = "/tmp/production_clone"
  website_source = tmp_clone + "/" + source_directory
  git_clone_branch(:production, source_repo, tmp_clone)

  files = get_file_list(website_source)

  keys = get_keys(aws_key_file_name)

  update_files(keys, website_production_bucket, website_source, files)

  puts "Remove #{temp_clone}"
  FileUtils.rm_rf tmp_clone

end

