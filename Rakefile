def rvm_ruby_dir
  @rvm_ruby_dir ||= File.expand_path('../..', `which ruby`.strip)
end

namespace :travis do
  task :install_opencflite_debs do
    sh "mkdir -p .debs"
    Dir.chdir(".debs") do
      base_url = "https://github.com/downloads/CocoaPods/OpenCFLite"
      %w{ opencflite1_248-1_i386.deb opencflite-dev_248-1_i386.deb }.each do |deb|
        sh "wget #{File.join(base_url, deb)}" unless File.exist?(deb)
      end
      sh "sudo dpkg -i *.deb"
    end
  end

  task :fix_rvm_include_dir do
    unless File.exist?(File.join(rvm_ruby_dir, 'include'))
      # Make Ruby headers available, RVM seems to do not create a include dir on 1.8.7, but it does on 1.9.3.
      sh "mkdir '#{rvm_ruby_dir}/include'"
      sh "ln -s '#{rvm_ruby_dir}/lib/ruby/1.8/i686-linux' '#{rvm_ruby_dir}/include/ruby'"
    end
  end

  task :setup => [:install_opencflite_debs, :fix_rvm_include_dir] do
    sh "CFLAGS='-I#{rvm_ruby_dir}/include' bundle install"
  end
end

desc "Run `pod spec lint` on all specs"
task :lint do
  exit if ENV['skip-lint']

  ENV['SKIP_SETUP']='1'
  ENV['CP_REPOS_DIR']= Pathname.new(Dir.pwd).dirname.to_s

  specs = `git diff-index --name-only HEAD | grep '.podspec$'`.strip.split("\n")
  specs = ['.'] if specs.empty?
  last_commit_podspecs = `git diff --diff-filter=ACMRTUXB --name-only HEAD~1..HEAD | grep '.podspec$'`.strip.split("\n")
  last_commit_specs = last_commit_podspecs.map {|p| p.gsub(/(.*)\/.*\/.*/,'\1')}.uniq

  failures = 0

  # unless last_commit_podspecs.empty?
  #   puts "\n>>> last commit podspecs (full lint) <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<\n\n"
  #   command = "pod spec lint '#{last_commit_podspecs.join("' '")}' "
  #   failures += 1 unless excute_command(command)
  # end

  unless last_commit_specs.empty?
    puts "\n>>> last commit pods (quick lint with warnings) <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<\n\n"
    command = "pod spec lint --quick '#{last_commit_specs.join("' '")}' "
    failures += 1 unless excute_command(command)
  end

  puts "\n>>> Repo <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<\n\n"
  command = "pod repo lint ."
  failures += 1 unless excute_command(command)

  unless failures.zero?
    exit 1
  end
end

def excute_command(command)
  # begin
    puts command
    # do it this way so we can trap Interrupt, doesn't work well with Kernel::system and Rake's sh
    system command
  # rescue Interrupt
  #   break
  #   false
end

task :default => :lint
