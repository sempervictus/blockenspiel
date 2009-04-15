# -----------------------------------------------------------------------------
# 
# Blockenspiel Rakefile
# 
# -----------------------------------------------------------------------------
# Copyright 2008-2009 Daniel Azuma
# 
# All rights reserved.
# 
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
# 
# * Redistributions of source code must retain the above copyright notice,
#   this list of conditions and the following disclaimer.
# * Redistributions in binary form must reproduce the above copyright notice,
#   this list of conditions and the following disclaimer in the documentation
#   and/or other materials provided with the distribution.
# * Neither the name of the copyright holder, nor the names of any other
#   contributors to this software, may be used to endorse or promote products
#   derived from this software without specific prior written permission.
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.
# -----------------------------------------------------------------------------

require 'rake'
require 'rake/clean'
require 'rake/gempackagetask'
require 'rake/testtask'
require 'rake/rdoctask'

require File.expand_path("#{File.dirname(__FILE__)}/lib/blockenspiel/version")


# Configuration
extra_rdoc_files_ = ['README.rdoc', 'HISTORY.rdoc', 'ImplementingDSLblocks.rdoc']


# Default task
if RUBY_PLATFORM =~ /java/
  task :default => [:clean, :compile_java, :rdoc, :test]
else
  task :default => [:clean, :compile, :rdoc, :test]
end


# Clean task
CLEAN.include(['ext/blockenspiel/Makefile', '**/unmixer.bundle', 'ext/blockenspiel/unmixer.o', '**/blockenspiel_unmixer.jar', 'ext/blockenspiel/BlockenspielUnmixerService.class', 'idslb_markdown.txt', 'doc', 'pkg'])


# Test task
Rake::TestTask.new('test') do |task_|
  task_.pattern = 'tests/tc_*.rb'
end


# RDoc task
Rake::RDocTask.new do |task_|
  task_.main = 'README.rdoc'
  task_.rdoc_files.include(*extra_rdoc_files_)
  task_.rdoc_files.include('lib/**/*.rb')
  task_.rdoc_dir = 'doc'
end


# Gem task
gemspec_ = Gem::Specification.new do |s_|
  s_.name = 'blockenspiel'
  s_.summary = 'Blockenspiel is a helper library designed to make it easy to implement DSL blocks.'
  s_.version = Blockenspiel::VERSION_STRING
  s_.author = 'Daniel Azuma'
  s_.email = 'dazuma@gmail.com'
  s_.description = 'Blockenspiel is a helper library designed to make it easy to implement DSL blocks. It is designed to be comprehensive and robust, supporting most common usage patterns, and working correctly in the presence of nested blocks and multithreading.'
  s_.homepage = 'http://virtuoso.rubyforge.org/blockenspiel'
  s_.rubyforge_project = 'virtuoso'
  s_.required_ruby_version = '>= 1.8.7'
  s_.files = FileList['ext/**/*.{c,rb}', '{lib,tests}/**/*.rb', '*.rdoc', 'Rakefile'].to_a
  s_.extra_rdoc_files = extra_rdoc_files_
  s_.has_rdoc = true
  if RUBY_PLATFORM =~ /java/
    s_.platform = 'jruby'
    s_.files += ['lib/blockenspiel_unmixer.jar']
  else
    s_.platform = Gem::Platform::RUBY
    s_.extensions = ['ext/blockenspiel/extconf.rb']
  end
end
Rake::GemPackageTask.new(gemspec_) do |task_|
  task_.need_zip = false
  task_.need_tar = false
end


# Build tasks for MRI
dlext_ = Config::CONFIG['DLEXT']

desc 'Builds the extension'
task :compile => ["lib/blockenspiel/unmixer.#{dlext_}"]

file 'ext/blockenspiel/Makefile' => ['ext/blockenspiel/extconf.rb'] do
  Dir.chdir('ext/blockenspiel') do
    ruby 'extconf.rb'
  end  
end

file "ext/blockenspiel/unmixer.#{dlext_}" => ['ext/blockenspiel/Makefile', 'ext/blockenspiel/unmixer.c'] do
  Dir.chdir('ext/blockenspiel') do
    sh 'make'
  end
end

file "lib/blockenspiel/unmixer.#{dlext_}" => ["ext/blockenspiel/unmixer.#{dlext_}"] do
  cp "ext/blockenspiel/unmixer.#{dlext_}", 'lib/blockenspiel'
end


# Build tasks for JRuby
desc "Compiles the JRuby extension"
task :compile_java do
  Dir.chdir('ext/blockenspiel') do
    sh 'javac -source 1.5 -target 1.5 -classpath $JRUBY_HOME/lib/jruby.jar BlockenspielUnmixerService.java'
    sh 'jar cf blockenspiel_unmixer.jar BlockenspielUnmixerService.class'
    cp 'blockenspiel_unmixer.jar', '../../lib/blockenspiel_unmixer.jar'
  end
end


# Custom task that takes the implementing dsl blocks paper
# and converts it from RDoc format to Markdown
task :idslb_markdown do
  File.open('ImplementingDSLblocks.txt') do |read_|
    File.open('idslb_markdown.txt', 'w') do |write_|
      linenum_ = 0
      read_.each do |line_|
        linenum_ += 1
        next if linenum_ < 4
        line_.sub!(/^===\ /, '### ')
        line_.sub!(/^\ \ /, '    ')
        if line_[0..3] == '### '
          line_.gsub!(/(\w)_(\w)/, '\1\_\2')
        end
        if line_[0..3] != '    '
          line_.gsub!('"it_should_behave_like"', '"it\_should\_behave\_like"')
          line_.gsub!('"time_zone"', '"time\_zone"')
          line_.gsub!(/\+(\w+)\+/, '`\1`')
          line_.gsub!(/\*(\w+)\*/, '**\1**')
          line_.gsub!(/<\/?em>/, '*')
          line_.gsub!(/<\/?tt>/, '`')
          line_.gsub!(/<\/?b>/, '**')
          line_.gsub!(/\{([^\}]+)\}\[([^\]]+)\]/) do |match_|
            text_, url_ = $1, $2
            "[#{text_.gsub('_', '\_')}](#{url_})"
          end
          line_.gsub!(/\ (http:\/\/[^\s]+)/, ' [\1](\1)')
        end
        write_.puts(line_)
      end
    end
  end
end
