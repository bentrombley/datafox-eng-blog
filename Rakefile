###
#  Author: Alex Zappa a.k.a. re[at]lat
#  Email: reatlat@gmail.com
#  URL: https://reatlat.net/
###

require "rubygems"
require "bundler/setup"
require "jekyll"
require "uuid"


class String
  def red;    "\e[1;31m#{self}\e[0m" end
  def blue;   "\e[1;34m#{self}\e[0m" end
  def green;  "\e[0;32m#{self}\e[0m" end
  def yellow; "\e[0;33m#{self}\e[0m" end
  def gray;   "\e[0;37m#{self}\e[0m" end
end


def say_what? message
  print message
  STDIN.gets.chomp
end


def sluggize str
  str.downcase.gsub(/[^a-z0-9]+/, '-').gsub(/^-|-$/, '');
end


def if_exit str
  if str == 'exit' || str == 'quit' || str == 'done'
    puts ""
    puts "   All done!" if str == 'done'
    puts "   See you soon =)"
    puts ""
    exit 1
  end
end


desc "Create a new post"
task :new do
  puts "  For create a new post enter [Category, Tags, Title]"
  puts "  For quit enter [exit] in anytime."
  puts ""
  category  = say_what?('  Category:  ')
  if category.to_s == ''
    puts ""
    puts "  Category can't be empty!".red
    puts ""
    Rake::Task["new"].reenable
    Rake::Task["new"].invoke
  end

  # exit if category == exit
  if_exit(category)

  tags      = say_what?('  Tags:      ')
  if_exit(tags)
  title     = say_what?('  Title:     ')
  if_exit(title)
                   puts '  Author:    '
  author    = say_what?('    name:    ')
  author = 'null' if author.to_s == ''
  if_exit(author)
  twitter     = say_what?('    twitter: ')
  twitter = 'null' if twitter.to_s == ''
  if_exit(twitter)
  github     = say_what?('    github:  ')
  github = 'null' if github.to_s == ''
  if_exit(github)
  filename  = "_drafts/#{Time.now.strftime('%Y-%m-%d')}-#{sluggize title}.md"

  if File.exist? filename
    puts ""
    puts "  Can't create new post: " + filename.yellow
    puts "    - Path already exists.".red
    puts ""
    Rake::Task["action"].reenable
    Rake::Task["action"].invoke
  end

  File.open(filename, "w+") do |post|
    post.puts "---"
    post.puts "layout:       post"
    post.puts "uuid:         #{UUID.new.generate}"
    post.puts "categories:   #{category}"
    post.puts "tags:         [#{tags}]"
    post.puts "title:        '#{title}'"
    post.puts "author:       "
    post.puts "  name:       #{author}"
    post.puts "  twitter:    #{twitter}"
    post.puts "  github:     #{github}"
    post.puts "feature_img:  null"
    post.puts "sitemap:"
    post.puts "  lastmod:    #{Time.now.strftime('%FT%T')}"
    post.puts "  priority:   0.5"
    post.puts "  changefreq: monthly"
    post.puts "  exclude:    'no'"
    post.puts "---"
    post.puts ""
    post.puts "Once upon a time..."
  end

  puts ""
  puts "  A new post was created for at:"
  puts "    " + filename.green
  if_exit('exit')

end


desc "Default task"
task :default do

  puts ""
  puts " ██████╗  █████╗ ████████╗ █████╗ ███████╗ ██████╗ ██╗  ██╗ ".yellow
  puts " ██╔══██╗██╔══██╗╚══██╔══╝██╔══██╗██╔════╝██╔═══██╗╚██╗██╔╝ ".yellow
  puts " ██║  ██║███████║   ██║   ███████║█████╗  ██║   ██║ ╚███╔╝  ".yellow
  puts " ██║  ██║██╔══██║   ██║   ██╔══██║██╔══╝  ██║   ██║ ██╔██╗  ".yellow
  puts " ██████╔╝██║  ██║   ██║   ██║  ██║██║     ╚██████╔╝██╔╝ ██╗ ".yellow
  puts " ╚═════╝ ╚═╝  ╚═╝   ╚═╝   ╚═╝  ╚═╝╚═╝      ╚═════╝ ╚═╝  ╚═╝ ".yellow
  puts ""
  puts "              Copyright © 2017 | MIT License               ".yellow
  puts ""
  Rake::Task["new"].reenable
  Rake::Task["new"].invoke

end

