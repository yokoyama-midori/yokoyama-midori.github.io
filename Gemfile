# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll-theme-chirpy", "~> 7.2", ">= 7.2.3"

gem "html-proofer", "~> 5.0", group: :test

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.2.0", :platforms => [:mingw, :x64_mingw, :mswin]

# [2025-02-11 17:54:02] ERROR NoMethodError: undefined method `serve' for an instance of Jekyll::Commands::Serve::Servlet
#         /home/y_midori/.rbenv/versions/3.3.5/lib/ruby/gems/3.3.0/gems/webrick-1.9.1/lib/webrick/httpservlet/filehandler.rb:242:in `service'
#         /home/y_midori/.rbenv/versions/3.3.5/lib/ruby/gems/3.3.0/gems/webrick-1.9.1/lib/webrick/httpserver.rb:140:in `service'
#         /home/y_midori/.rbenv/versions/3.3.5/lib/ruby/gems/3.3.0/gems/webrick-1.9.1/lib/webrick/httpserver.rb:96:in `run'
# /home/y_midori/.rbenv/versions/3.3.5/lib/ruby/gems/3.3.0/gems/webrick-1.9.1/lib/webrick/server.rb:310:in `block in start_thread'
# ↓付けたら治った
gem "webrick", "~> 1.7.0"