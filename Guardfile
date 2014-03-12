require 'colorize'
require 'pry'

HOST          = "http://ranmocy.info/"
SITE_PATH     = File.expand_path('./.site')    # need to be absolute
BUILD_PATH    = File.expand_path('/tmp/brain') # need to be absolute
LAYOUT_PATH   = File.expand_path('./.layout') # need to be absolute
CATEGORIES    = ['Blog', 'Diary', 'Dream', 'Idea', 'Org', 'Philosophy', 'Piece', 'Poem', 'Remark', 'Tech', 'Translation', 'Young']

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

  def scan_file(path, file_path) # path for url
    file          = Hashie::Mash.new
    file.ext      = File.extname(file_path)
    file.src_path = file_path
    file.url      = file_path[path.length..-1]
    file.content  = File.read(file_path)

    # read yaml header if possible
    begin
      header       = file.content.match /\A---\n(.*)\n---\n\n(.*)\Z/m
      file.meta    = Hashie::Mash.new header ? YAML.load(header[1]) : nil
      file.content = header[2] if header # remove the header
    rescue ArgumentError => e # images are not UTF-8
      file.meta = Hashie::Mash.new
    end
    file.meta.category = (file.meta.category || "default").downcase

    # url
    case file.ext
    when ".slim", ".scss"
      file.url = remove_ext(file.url)
    when ".md"
      file.url = replace_ext(file.url, ".html")
    end
    # move file.html to file/index.html for url
    if (File.extname(file.url) == '.html') && File.basename(file.url) != 'index.html'
      file.url = replace_ext(file.url, '/')
      file.dest_path = File.join(BUILD_PATH, file.url, 'index.html')
    else
      file.dest_path = File.join(BUILD_PATH, file.url)
    end
    file.url.downcase!

    file
  end

  def scan(dir_path, root: nil)
    Dir.glob("#{dir_path}/**/*").reject { |path| File.directory?(path) }
    .collect { |file_path| scan_file(root || dir_path, file_path) }
  end

  def self.files
    @@files
  end

  def self.layouts
    @@layouts
  end

  def self.articles
    @@articles
  end

  def self.articles_by_category
    @@articles_by_category
  end

  def self.categories
    @@categories
  end

  def call(guard_class, event, *args)
    @@files    = scan(SITE_PATH)
    @@layouts  = scan(LAYOUT_PATH).inject({}) do |layouts, file|
      layouts[File.basename(file.src_path, ".html.slim")] = file
      layouts
    end
    @@articles = (Dir.glob('*').select { |d| File.directory?(d) } - ["draft", "en"]).map { |category|
      scan(File.expand_path(category), root: File.expand_path('.')).each { |file| file.category = category }
    }.flatten
    .sort { |a,b| [a.meta['created-at'], a.meta['updated-at']] <=> [b.meta['created-at'], b.meta['updated-at']] }
    .reverse

    h = Hash.new([])
    h["memories"] = @@articles
    @@articles_by_category = @@articles.inject(h) { |h, article|
      h[article.meta.category] += [article]; h
    }

    layout_content = File.read(File.join(LAYOUT_PATH, 'category.html.slim'))
    @@categories = @@articles_by_category.map do |name, articles|
      Hashie::Mash.new({
        url: "/#{name}/",
        content: layout_content,
        src_path: name,
        dest_path: File.join(BUILD_PATH, "/#{name}/index.html"),
        meta: Hashie::Mash.new({
          category: name.downcase,
          articles: articles,
          }),
        })
    end

    # FIXME: motto is on _aside but not a category
    @@articles_by_category["motto"] = File.read(File.join(SITE_PATH, 'motto.html.slim')).scan(/^ +li .*$/)
  end

end

class SlimEnv

  def initialize(file=nil)
    metaclass = class << self; self; end
    metaclass.send(:define_method, :current_page) { file }
    @settings = Hashie::Mash.new({
      title: "Ranmocy's Fragments",
      description: "My Brain, My Treasure.",
      author: "Ranmocy",
      email: "Ranmocy@gmail.com",
      host: "http://ranmocy.info/",
      categories: CATEGORIES,
      })
  end

  def settings
    @settings
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

  def categories
    Scanner.articles_by_category
  end

  def groups
    h = {life: ["blog", "diary", "dream", "poem", ],
      idea: ["motto", "idea", "remark", "philosophy", ],
      work: ["tech", "piece", "translation"], }
  end

  def motto
    Hashie::Mash.new({
      blog: "旅行日志",
      diary: "一个欲望灼烧者艰难写下的自白。",
      dream: "最真实总是梦境",
      poem: "用诗歌来拯救自我",
      motto: "一句话评点世界",
      idea: "思维碎片，漂浮在名叫头脑的海洋。",
      remark: "从这个世界剥离出的抽象",
      philosophy: "我在教导你们世界运行的原动力。你们听之，想之，就忘之吧。",
      tech: "技术宅拯救世界。",
      piece: "What I did define what I am.",
      translation: "Words worth spreading widely.",
      memories: "三千竹水，不生不灭",
    })
  end

  def active? category
    category == current_page.meta.category ? "active" : nil
  end

  def method_missing(meth, *args, &blk)
    if current_page && current_page.meta.include?(meth.to_s)
      current_page.meta[meth.to_s]
    else
      super
    end
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

  # cache of layouts
  def layout(name)
    Slim::Template.new { Scanner.layouts[name.to_s].content }
  end

  def apply_layout(file, helper, content, default_layout: "default")
    layout_name = file.meta["layout"] || default_layout
    puts "render layout #{layout_name} for #{file.url}".yellow
    res = layout(layout_name).render(helper) { content }
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

  def generate_markdown(file)
    # TODO: generate by category
    apply_layout file, SlimEnv.new(file), RDiscount.new(file.content).to_html, default_layout: "article"
  end

  def call(guard_class, event, *args)
    cleanup

    Scanner.files.each do |file|
      cmd = "generate_#{file.ext[1..-1]}"
      if respond_to? cmd
        puts "#{file.ext[1..-1].upcase}: #{file.src_path}"
        self.send cmd, file
      else
        puts "copying #{file.url} to #{file.dest_path}".green
        copy_file(file.src_path, file.dest_path)
      end
    end

    Scanner.articles.each do |article|
      generate_markdown(article)
    end

    Scanner.categories.each do |file|
      helper = SlimEnv.new(file)
      apply_layout file, helper, Slim::Template.new { file.content }.render(helper)
    end
  end

end

guard :shell do
  # watch(/(.*)/) {|m| n "=> #{m[1]} ", "Brain", :success; m[1]} # TODO: setup livereload
  callback(Server.new, [:start_end, :stop_end])
  callback(Scanner.new, [:start_begin, :run_all_begin, :run_on_modifications_begin])
  callback(Generator.new, [:start_end, :run_all_end, :run_on_modifications_end])
  # callback([:start_begin, :run_on_modifications_begin]) { SlimEnv.reset }
end
