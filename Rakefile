require 'fileutils'
require 'zip'

target_versions = %w[0.43.05 0.44.02]
mod_directories = %w[Evil_Dwarves Norse Savage_Dwarves]

desc 'merge all mod branches into master branches'
task :merge_mods do
  mod_branches = `git branch`
                 .split("\n")
                 .select { |branch| /mod\//.match branch }
                 .map { |branch| branch.delete('*') }
                 .map { |branch| branch.gsub(/\s+/, '') }

  puts "Will merge: #{mod_branches.join(', ')}"
  on_version_branches do |version|
    puts "----- Merging mods for version #{version} -----"

    mod_branches.each do |mod|
      mod_head = `git log -1 --format="%h" #{mod}`
      sh "git merge --squash #{mod}"
      sh "git commit -m \"Merge #{mod} (#{mod_head.chomp})\""
    end
  end
end

desc 'release zips of the different combinations of mods possible'
task :release_zips do
  on_version_branches do |version|
    FileUtils.mkdir_p 'target'
    mod_count = mod_directories.length
    (1..mod_count).each do |i|
      puts "----- Generating #{i}-mod combination(s)"
      mod_directories.combination(i).to_a.each do |combination|
        FileUtils.mkdir_p 'temp'
        puts "  `-- #{combination.join(', ')}"
        combination.each do |directory|
          FileUtils.cp_r(directory + '/.', 'temp')
        end

        zipfile_name = "target/niezbop_#{combination.join('_')}_#{version}.zip"
        File.delete zipfile_name if File.file? zipfile_name
        Zip::File.open(zipfile_name, Zip::File::CREATE) do |zipfile|
          Dir['temp/**/[^README]*.[^ico]*'].each do |file|
            # Two arguments:
            # - The name of the file as it will appear in the archive
            # - The original file, including the path to find it
            zipfile.add(file.gsub(/temp\//, ''), file)
          end
        end
        FileUtils.remove_dir 'temp'
      end
    end
  end
end

def on_version_branches
  temp = current_branch
  target_versions.each do |version|
    branch = "master.#{version}"

    begin
      sh "git rev-parse --verify #{branch}"
    rescue
      puts "Branch #{branch} does not exist"
      next
    end

    sh "git checkout #{branch}"

    yield version
  end
  sh "git checkout #{temp}"
end

def current_branch
  `git rev-parse --abbrev-ref HEAD`
end
