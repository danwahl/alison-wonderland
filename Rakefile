namespace :book do
  def exec_or_raise(command)
    puts `#{command}`
    if !$?.success?
      raise "'#{command}' failed"
    end
  end

  # Variables referenced for build
  version_string = `git describe --tags`.chomp
  if version_string.empty?
    version_string = "0"
  end
  date_string = Time.now.strftime("%Y-%m-%d")
  params = "--attribute revnumber='#{version_string}' --attribute revdate='#{date_string}'"

  desc "build basic book formats"
  task build: [:build_html, :build_epub, :build_pdf] do
    # Run check
    Rake::Task["book:check"].invoke

    # Rescue to ignore checking errors
  rescue => e
    puts e.message
    puts "Error when checking books (ignored)"
  end

  desc "build basic book formats (for ci)"
  task ci: [:build_html, :build_epub, :build_pdf] do
    # Run check, but don't ignore any errors
    Rake::Task["book:check"].invoke
  end

  desc "build HTML format"
  task :build_html do
    puts "Converting to HTML..."
    `bundle exec asciidoctor #{params} -a data-uri wonderland-inc.adoc`
    puts " -- HTML output at wonderland-inc.html"
  end

  desc "build Epub format"
  task :build_epub do
    puts "Converting to EPub..."
    `bundle exec asciidoctor-epub3 #{params} wonderland-inc.adoc`
    puts " -- Epub output at wonderland-inc.epub"
  end

  desc "build PDF format"
  task :build_pdf do
    puts "Converting to PDF... (this one takes a while)"
    `bundle exec asciidoctor-pdf #{params} wonderland-inc.adoc 2>/dev/null`
    puts " -- PDF output at wonderland-inc.pdf"
  end

  desc "Check generated books"
  task check: [:build_html, :build_epub] do
    puts "Checking generated books"

    exec_or_raise("htmlproofer .")
    exec_or_raise("epubcheck wonderland-inc.epub")
  end

  desc "Clean all generated files"
  task :clean do
    puts "Removing generated files"

    FileList["wonderland-inc.html", "wonderland-inc.epub", "wonderland-inc.pdf"].each do |file|
      rm file

      # Rescue if file not found
    rescue Errno::ENOENT => e
      begin
        puts e.message
        puts "Error removing files (ignored)"
      end
    end
  end
end

task default: "book:build"
