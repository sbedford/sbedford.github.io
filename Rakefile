require "html-proofer"

task default: :test

task :build do
  sh "bundle exec jekyll build"
end

task test: :build do
  HTMLProofer.check_directory("./_site", {
    disable_external: true,
    checks: ["Links", "Images", "Scripts"],
    allow_hash_href: true,
    ignore_urls: [/feeds\.feedburner\.com/],
  }).run
end
