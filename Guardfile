require 'colorize'
require "pry"

SITE_PATH   = File.expand_path('.site')
BUILD_PATH  = File.expand_path('/tmp/brain')
LAYOUT_PATH = File.expand_path('.layouts')
ARTICLE_PATH = File.expand_path('posts')

class Server
  require "webrick"
  include WEBrick

  def call(guard_class, event, *args)
    case event
    when :start_end
      log_file = File.open('/tmp/brain.log', 'a+')
      @server = HTTPServer.new(DocumentRoot: BUILD_PATH,
                               Port: 8080,
                               AccessLog: [[log_file, AccessLog::COMBINED_LOG_FORMAT]],
                               Logger: Log.new(log_file))
      Thread.new { @server.start }     # Let's Rock!
    when :stop_end
      @server.shutdown                 # shutdown with guard
    end
  end
end

class Scanner
  require 'hashie'
  require 'yaml'

  def remove_ext(file)
    File.join(File.dirname(file), File.basename(file, File.extname(file)))
  end

  def replace_ext(file, ext)
    File.join(File.dirname(file), File.basename(file, File.extname(file)) + ext)
  end

  def scan_file(file_path, root: File.expand_path('.')) # path for url
    file          = Hashie::Mash.new
    file.ext      = File.extname(file_path)
    file.src_path = file_path
    file.content  = File.read(file_path)

    # read yaml header if possible
    begin
      header       = file.content.match(/\A---\n(.*)\n---\n\n(.*)\Z/m)
      file.meta    = Hashie::Mash.new header ? YAML.load(header[1]) : nil
      file.content = header[2] if header # remove the header
    rescue ArgumentError # images are not UTF-8
      file.meta = Hashie::Mash.new
    end

    # url
    file.url = file_path[root.size..-1]
    case file.ext
    when ".slim", ".scss"
      file.url = remove_ext(file.url)
    when ".md"
      file.url = replace_ext(file.url, ".html")
    end
    file.dest_path = File.join(BUILD_PATH, file.url)

    file
  end

  def scan(dir_path)
    Dir.glob("#{dir_path}/**/*").select { |path| File.file?(path) }
    .map { |file_path| scan_file(file_path, root: dir_path) }
  end

  def call(guard_class, event, *args)
    @@files = scan(SITE_PATH)
    # Layouts are `layout_name => layout_file`
    @@layouts = scan(LAYOUT_PATH).inject({}) do |layouts, file|
      layouts[File.basename(file.src_path, ".html.slim")] = file
      layouts
    end
    @@articles = scan(ARTICLE_PATH).sort_by { |a| a.meta.created_at }.reverse
  end

  def self.method_missing(meth, *args, &blk)
    class_variable_defined?("@@#{meth}") ? class_variable_get("@@#{meth}") : super
  end

end

class SlimEnv

  def initialize(file=nil)
    metaclass = class << self; self; end
    metaclass.send(:define_method, :current_page) { file }
  end

  def articles
    Scanner.articles
  end

  def include(name, options = {}, &block)
    Slim::Template.new { Scanner.layouts[name.to_s].content }.render(self, &block)
  end

  def partial(name, options = {}, &block)
    include("_#{name}", options, &block)
  end

  def options_str options
    options.map { |k,v| "#{k}=\"#{v}\"" }.join(" ")
  end

  def link_to name, url, options = {}
    "<a href='#{url}' #{options_str(options)}>#{name}</a>"
  end

  def image_tag name, options = {}
    "<img src='/assets/images/#{name}' #{options_str(options)} />"
  end

  def stylesheet_link_tag name
    "<link href=\"/assets/stylesheets/#{name}.css\" media=\"screen\" rel=\"stylesheet\" type=\"text/css\">"
  end

  def javascript_include_tag name
    "<script src=\"/assets/javascripts/#{name}.js\" type=\"text/javascript\"></script>"
  end

  def method_missing(meth, *args, &blk)
    current_page.meta.send(meth.to_s) # return nil if no method
  end

end

class Generator
  require 'slim'
  require 'rdiscount'
  require "fileutils"

  def cleanup
    FileUtils.rm_r(BUILD_PATH, secure: true) if File.exists? BUILD_PATH
    FileUtils.mkdir_p(BUILD_PATH)
  end

  def copy_file(src_path, dest_path)
    FileUtils.mkdir_p File.dirname(dest_path)
    FileUtils.cp src_path, dest_path
  end

  def write_file(dest_path, content)
    FileUtils.mkdir_p File.dirname(dest_path)
    File.open(dest_path, "w") { |f| f.write(content) }
  end

  def apply_layout(file, helper, content, default_layout: "default")
    name = file.meta.layout || default_layout
    puts "render layout #{name} for #{file.url}".yellow
    layout = Slim::Template.new { Scanner.layouts[name.to_s].content }
    res = layout.render(helper) { content }
    write_file(file.dest_path, res)
  end

  def generate_slim(file)
    helper = SlimEnv.new(file)
    res    = Slim::Template.new { file.content }.render(helper)
    apply_layout(file, helper, res)
  end

  def generate_scss(file)
    FileUtils.mkdir_p File.dirname(file.dest_path)
    res = `scss #{file.src_path} #{file.dest_path} 2>&1`
    puts res.red unless $?.success?
  end

  def generate_md(file)
    apply_layout file, SlimEnv.new(file), RDiscount.new(file.content).to_html
  end

  def call(guard_class, event, *args)
    cleanup
    (Scanner.files + Scanner.articles).each do |file|
      cmd = "generate_#{file.ext[1..-1]}"
      if respond_to? cmd # I know how to deal with it
        puts "#{file.ext[1..-1].upcase}: #{file.url}"
        self.send cmd, file
      else # No, I don't know, just copy it.
        puts "copying #{file.url}".green
        copy_file(file.src_path, file.dest_path)
      end
    end
  end

end

guard :shell do
  watch(/(.*\.(slim|scss|md))$/) {|m| n "=> #{m[1]} ", "Brain", :success; m[1]}
  callback(Server.new, [:start_end, :stop_end])
  callback(Scanner.new, [:start_begin, :run_all_begin, :run_on_modifications_begin])
  callback(Generator.new, [:start_end, :run_all_end, :run_on_modifications_end])
end
