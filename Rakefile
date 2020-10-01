require 'rubygems'
%w[maruku erubis].each do |dep|
  require dep rescue puts "Stat requires the #{dep} Gem. `gem install #{dep}` to continue..." and exit
end

ENV['STAT_REMOTE'] = "deployer@mulva:/var/www/test"

task :default => "stat:get_started"
namespace :stat do
  desc "Get Started!"
  task :get_started do
    mkdir_p "src/stylesheets"
    mkdir_p "src/images"
    mkdir_p "src/javascripts"
    md,html = File.read(__FILE__).split(/^__END__$/)[1].split(/^=+$/).map {|s| s.strip}
    File.open('src/layout.html.erb', 'w') { |f| f.puts html }
    File.open('src/readme.md', 'w') { |f| f.puts md }
    Rake::Task["stat:build"].invoke
  end
  
  task :initialize do
    mkdir_p "pkg"
  end
  
  FileList['src/*.md'].each do |src|
    output = src.sub(/src\/(.+).md/, 'pkg/\1.html')
    action = $1
    layout = "src/layout.html.erb"
    file output => [src, layout] do
      content = Maruku.new(File.read(src)).to_html
      built = Erubis::Eruby.new(File.read(layout)).evaluate({:content => content, :action => action})
      File.open(output, 'w') { |f| f.puts built }
    end
    task :build => [:initialize, output]
  end
  
  FileList['src/stylesheets/*', 'src/images/*', 'src/javascripts/*'].each do |src|
    output = src.sub(/^src/,'pkg')
    file output => src do
      mkdir_p 'pkg/'+src.split('/')[1]
      cp src, output
    end
    task :build => [:initialize, output]
  end
  
  desc "Build the site"
  task :build
  
  desc "Force a full rebuild"
  task :rebuild => [:clean, :build]
  
  desc "Clean the pkg directory"
  task :clean do
    rm_rf "pkg"
  end
  
  desc "Build & Resync to the configured host"
  task :deploy => :build do
    puts ENV['STAT_REMOTE'] ? `rsync -avz -e ssh pkg/ #{ENV['STAT_REMOTE']}` : "Usage: rake stat:deploy STAT_REMOTE=user@host:path"
  end
end

__END__
## Stat ##
The joy of simple static site maintenance made super easy.

Hopefully you've already run `rake stat:get_started` from the directory containing the Rakefile, if not, go for it!  The idea is simple.  Define the content of your web pages using markdown, and slap a common layout around them.  Only rebuilding or uploading files when neccesary.  

From within `src/layout.html.erb` you have access to two variables, `@content` and `@action`.  `@content` is what was generated for any given `*.md` file in your src directory and `@action` is the `*` part of the `*.md` file _use this for adding a class to navigation links, see `src/layout.html.erb`_.  Set the constant at the top of the Rakefile to the appropriate value if you intend to rsync the generated html somewhere.

Happy Stating!

=======================================

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
	"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
  <link rel="stylesheet" href="stylesheets/screen.css" type="text/css" media="screen" title="no title" charset="utf-8">
	<title>Stat</title>
</head>
<body>
  <div id="header">
    <ul id="menu">
      <%- ["readme", "noexist"].each do |a| -%>
        <%= "<li><a href='#{a}.html' class='#{'cur' if @action == a}'>#{a.capitalize}</a></li>" %>
      <%- end -%>
    </ul>
  </div>
  <div id="content">
  	<%= @content %>
  </div>
</body>
</html>
