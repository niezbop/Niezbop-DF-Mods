target_versions = %w[0.43.05 0.44.02]

desc 'merge all mod branches into master branches'
task :merge_mods do
  mod_branches = `git branch`
			  .split("\n")
			  .select { |branch| /mod\//.match branch }
			  .map { |branch| branch.gsub(/\*/, '') }
			  .map { |branch| branch.gsub(/\s+/, '') }

  puts "Will merge: #{mod_branches.join(', ')}"
  target_versions.each do |version|
	puts "----- Merging mods for version #{version} -----"
	branch = "master.#{version}"
	begin
	  sh "git rev-parse --verify #{branch}"
	rescue
	  puts "Branch #{branch} does not exist yet"
	  next
	end

	sh "git checkout #{branch}"

	mod_branches.each do |mod|
	  mod_head = `git log -1 --format="%h" #{mod}`
	  sh "git merge --squash #{mod}"
	  sh "git commit -m \"Merge #{mod} (#{mod_head.chomp})\""
	end
  end
end