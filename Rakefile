require "bundler/gem_tasks"
require "shell"

c_source = "../c/meterpreter/"
java_source = "../java"
php_source = "../php/meterpreter/"
python_source = "../python/meterpreter/"
dest = "./data"
meterpreter_dest = "./data/meterpreter"

platform_config = {
  :windows => {
    :sources => [
      "../c/meterpreter/output/x64",
      "../c/meterpreter/output/x86"
    ],
    :extensions => [
      "dll"
    ]
  },
  :posix => {
    :sources => [
      "../c/meterpreter/data/meterpreter"
    ],
    :extensions => [
      "lso", "bin"
    ]
  },
  :java => {
    :sources => [
      "../java/output/data/meterpreter"
    ],
    :extensions => [
      "jar"
    ],
  },
  :php => {
    :sources => [
      php_source
    ],
    :extensions => [
      "php"
    ]
  },
  :python => {
    :sources => [
      python_source
    ],
    :extensions => [
      "py"
    ]
  }
}

def copy_files(cnf, meterpreter_dest)
  cnf[:sources].each do |f|
    cnf[:extensions].each do |ext|
      Dir.glob("#{f}/*.#{ext}").each do |bin|
        target = File.join(meterpreter_dest, File.basename(bin))
        print("Copying: #{bin} -> #{target}\n")
        FileUtils.cp(bin, target)
      end
    end
  end
end

task :create_dir do
  Dir.mkdir(dest) unless Dir.exist?(dest)
  Dir.mkdir(meterpreter_dest) unless Dir.exist?(meterpreter_dest)
end

task :win_compile do
  Dir.chdir(c_source) do
    system('cmd.exe /c make.bat')
  end
end

task :posix_compile do
  Dir.chdir(c_source) do
    system('make clean && make')
  end
end

task :java_compile do
  Dir.chdir(java_source) do
    system('mvn package -Ddeploy.path=output -Dandroid.release=true -q -P deploy')
  end
end

task :win_copy do
  copy_files(platform_config[:windows], meterpreter_dest)
end

task :posix_copy do
  copy_files(platform_config[:posix], meterpreter_dest)
end

task :java_copy do
  copy_files(platform_config[:java], meterpreter_dest)
  FileUtils.remove_entry_secure('./java', :force => true)
  FileUtils.cp_r('../java/output/data/java', dest)
end

task :php_copy do
  copy_files(platform_config[:php], meterpreter_dest)
end

task :python_copy do
  copy_files(platform_config[:python], meterpreter_dest)
end

task :win_prep => [:create_dir, :win_compile, :win_copy] do
end

task :posix_prep => [:create_dir, :posix_compile, :posix_copy] do
end

task :java_prep => [:create_dir, :java_compile, :java_copy] do
end

task :php_prep => [:create_dir, :php_copy] do
end

task :python_prep => [:create_dir, :python_copy] do
end

task :default => [:python_prep, :php_prep, :java_prep, :posix_prep] do
end

# Override tag_version in bundler-#.#.#/lib/bundler/gem_helper.rb to force signed tags
module Bundler
  class GemHelper
    def tag_version
      sh "git tag -m \"Version #{version}\" -s #{version_tag}"
      Bundler.ui.confirm "Tagged #{version_tag}."
      yield if block_given?
    rescue
      Bundler.ui.error "Untagging #{version_tag} due to error."
      sh_with_code "git tag -d #{version_tag}"
      raise
    end
  end
end
