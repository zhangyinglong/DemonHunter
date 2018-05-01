#!/usr/bin/ruby
# -*- coding: UTF-8 -*-

task :default => :new

# 创建新文章
require 'fileutils'
task :new do
	puts "请输入要创建的文章主题："
	@article = STDIN.gets.chomp
	puts "请输入文章标题："
	@name = STDIN.gets.chomp
	puts "请输入文章分类，以空格分隔："
	@categories = STDIN.gets.chomp
	@slug = "#{@article}"
	@slug = @slug.downcase.strip.gsub(' ', '-')
	@date = Time.now.strftime("%F")
	@post_name = "_posts/#{@date}-#{@slug}.markdown"
	@author = "zyl04401"

	if File.exist?(@post_name)
		desc "文章已存在"
	else
		FileUtils.touch(@post_name)
		open(@post_name, 'a') do |file|
			file.puts "---"
			file.puts "layout:     post"
			file.puts "title:      \"#{@name}\""
			file.puts "date:       #{Time.now}"
			file.puts "author:     \"#{@author}\""
			file.puts "categories: #{@categories}"
			file.puts "published: false"
			file.puts "---"
		end
	end
	exec "/Applications/MacDown.app/Contents/MacOS/MacDown #{@post_name}&"
end

# htmlproofer 检测链接是否正常
task :check do
	exec "bundle exec jekyll build && bundle exec htmlproofer _site"
end
