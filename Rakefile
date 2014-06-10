# Bootstrap task
#-----------------------------------------------------------------------------#

desc 'Install dependencies'
task :bootstrap do
  if system('which bundle')
    sh "bundle install"
  else
    $stderr.puts "\033[0;31m" \
      "[!] Please install the bundler gem manually:\n" \
      '    $ [sudo] gem install bundler' \
      "\e[0m"
    exit 1
  end
end

begin

  task :build do
    title "Building the gem"
  end

  require "bundler/gem_tasks"

  # Release tasks
  #-----------------------------------------------------------------------------#

  desc "Build the gem for distribution"
  task :release_build => ['ext:clean', 'ext:precompile', :build]

  desc "Runs the tasks necessary for the release of the gem"
  task :pre_release do
    title "Running pre-release tasks"
    tmp = File.expand_path('../tmp', __FILE__)
    sh "rm -rf '#{tmp}'"
    Rake::Task[:release_build].invoke
  end

  # Always prebuilt for gems!
  Rake::Task[:build].enhance [:pre_release]

  # Ext Namespace
  #-----------------------------------------------------------------------------#

  namespace :ext do
    desc "Clean the ext files"
    task :clean do
      title "Cleaning extension"
      sh "cd ext/xcodeproj && rm -f Makefile *.o *.bundle prebuilt/**/*.o prebuilt/**/*.bundle"
    end

    desc "Build the ext"
    task :build do
      title "Building the extension"
      Dir.chdir 'ext/xcodeproj' do
        if on_rvm?
          sh "CFLAGS='-I#{rvm_ruby_dir}/include' ruby extconf.rb"
        else
          sh "ruby extconf.rb"
        end
        sh "make"
      end
    end

    desc "Pre-compile the ext for default Ruby on 10.8 and 10.9"
    task :precompile do
      title "Pre-compiling the extension"
      versions = Dir.glob(File.expand_path('../ext/xcodeproj/prebuilt/*darwin*', __FILE__)).sort
      versions.each do |version|
        Dir.chdir version do
          subtitle "#{File.basename(version)}"
          sh "make"
        end
      end
    end

    desc "Clean and build the ext"
    task :cleanbuild => [:clean, :build]
  end

  # Travis support
  def on_rvm?
    `which ruby`.strip.include?('.rvm')
  end

  def rvm_ruby_dir
    @rvm_ruby_dir ||= File.expand_path('../..', `which ruby`.strip)
  end

  #-----------------------------------------------------------------------------#

  namespace :spec do
    desc "Run all specs"
    task :all do
      puts "\n\033[0;32mUsing #{`ruby --version`.chomp}\033[0m"
      Rake::Task["ext:cleanbuild"].invoke

      title "Running the specs"
      ENV['GENERATE_COVERAGE'] = 'true'
      sh "bundle exec bacon #{FileList['spec/**/*_spec.rb'].join(' ')}"
    end

    desc "Automatically run specs"
    task :kick do
      exec "bundle exec kicker -c"
    end

    desc "Run single spec"
    task :single, :spec_file do |t, args|
      sh "bundle exec bacon #{args.spec_file}"
    end
  end

  desc "Run all specs"
  task :spec => 'spec:all'

  task :default => :spec

rescue LoadError
  $stderr.puts "\033[0;31m" \
    '[!] Some Rake tasks haven been disabled because the environment' \
    ' couldn’t be loaded. Be sure to run `rake bootstrap` first.' \
    "\e[0m"
end

# UI Helpers
#-----------------------------------------------------------------------------#

# Prints a title.
#
def title(string)
  puts
  puts yellow(string)
  puts "-" * 80
end

# Prints a subtitle
#
def subtitle(string)
  puts cyan(string)
end

# Colorizes a string to yellow.
#
def yellow(string)
  "\033[0;33m#{string}\e[0m"
end

# Colorizes a string to cyan.
#
def cyan(string)
  "\n\033[0;36m#{string}\033[0m"
end
