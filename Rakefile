# use pluginized rpsec if it exists
rspec_base = File.expand_path(File.dirname(__FILE__) + '/../rspec/lib')
$LOAD_PATH.unshift(rspec_base) if File.exist?(rspec_base) and !$LOAD_PATH.include?(rspec_base)

require 'spec/rake/spectask'
require 'spec/rake/verify_rcov'

PluginName = 'inherit_views'

task :default => :spec

desc "Run the specs for #{PluginName}"
Spec::Rake::SpecTask.new(:spec) do |t|
  t.spec_files = FileList['spec/**/*_spec.rb']
  t.spec_opts  = ["--colour"]
end

desc "Generate RCov report for #{PluginName}"
Spec::Rake::SpecTask.new(:rcov) do |t|
  t.spec_files  = FileList['spec/**/*_spec.rb']
  t.rcov        = true
  t.rcov_dir    = 'doc/coverage'
  t.rcov_opts   = ['--text-report', '--exclude', "spec/,rcov.rb,#{File.expand_path(File.join(File.dirname(__FILE__),'../../..'))}"] 
end

namespace :rcov do
  desc "Verify RCov threshold for #{PluginName}"
  RCov::VerifyTask.new(:verify => :rcov) do |t|
    t.threshold = 100.0
    t.index_html = File.join(File.dirname(__FILE__), 'doc/coverage/index.html')
  end
end

# the following optional tasks are for CI and doc building
begin
  require 'hanna/rdoctask'
  require 'garlic/tasks'
  require 'grancher/task'
  
  task :cruise => ['garlic:all', 'doc:publish']
  
  Rake::RDocTask.new(:doc) do |d|
    d.options << '--all'
    d.rdoc_dir = 'doc'
    d.main     = 'README.rdoc'
    d.title    = "#{PluginName} API docs"
    d.rdoc_files.include('README.rdoc', 'History.txt', 'License.txt', 'Todo.txt', 'lib/**/*.rb')
  end

  namespace :doc do
    task :publish => :doc do
      Rake::Task['doc:push'].invoke unless uptodate?('.git/refs/heads/gh-pages', 'doc')
    end
    
    Grancher::Task.new(:push) do |g|
      g.keep_all
      g.directory 'doc', 'doc'
      g.branch = 'gh-pages'
      g.push_to = 'origin'
    end
  end

rescue LoadError
end

begin
  require 'rake/gempackagetask'
  $LOAD_PATH.unshift File.dirname(__FILE__) + '/lib'
  require 'inherit_views/version'

  spec = Gem::Specification.new do |s|
    s.name          = "inherit_views"
    s.version       = InheritViews::Version::String
    s.summary       = "Allow rails controllers to inherit views."
    s.description   = "Allow rails controllers to inherit views."
    s.author        = "Ian White"
    s.email         = "ian.w.white@gmail.com"
    s.homepage      = "http://github.com/ianwhite/inherit_views/tree"
    s.has_rdoc      = true
    s.rdoc_options << "--title" << "Pickle" << "--line-numbers"
    s.test_files    = FileList["spec/**/*_spec.rb"]
    s.files         = FileList["lib/**/*.rb", "License.txt", "README.rdoc", "Todo.txt", "History.txt"]
  end

  Rake::GemPackageTask.new(spec) do |p|
    p.gem_spec = spec
    p.need_tar = true
    p.need_zip = true
  end

  desc "Generate inherit_views.gemspec file"
  task :build do
    File.open('inherit_views.gemspec', 'w') {|f| f.write spec.to_ruby }
  end
rescue LoadError
end