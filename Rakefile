require 'yarrow'
require 'yarrow/output/context'
require 'yarrow/tools/output_file'
require 'rom'
require 'mustache'
require 'kramdown'

ROM.use :auto_registration

ROM.setup(:memory)

class Posts < ROM::Relation[:memory]
  def by_name(name)
    where(name: name)
  end
end

class PostsMapper < ROM::Mapper
  relation :posts
  model name: 'Post'

  attribute :name
  attribute :title
  attribute :body
  attribute :published

  symbolize_keys true
end

class CreatePost < ROM::Commands::Create[:memory]
  register_as :create
  relation :posts
  result :one
end

$rom = ROM.finalize.env

class SiteContext < Yarrow::Output::Context
  def document_title
    "#{post.title} | Changing Software"
  end

  def site_logo
    'Changing Software'
  end
end

def render_post(post)
 context = SiteContext.new(:post => post)
 template = File.read('templates/post.html')

 Mustache.render(template, context)
end

def write_file(path, content)
  document = Yarrow::Tools::OutputFile.new
  document.write(path, content)
end

include Yarrow::Tools::FrontMatter

task :collect do
  Dir['content/posts/*.md'].each do |path|
    content, data = extract_split_content(File.read(path, :encoding => 'utf-8'))
    name = File.basename(path, '.md')
    body = Kramdown::Document.new(content).to_html

    post = {
      name: name,
      title: data['title'],
      published: data['published'],
      body: body
    }

    $rom.command(:posts).create.call(post)
  end
end

task build: :collect do
  $rom.relation(:posts).as(:posts).each do |post|
    write_file("/posts/#{post.name}/", render_post(post))
  end
end

task :server do
  Yarrow::Configuration.register('./Yarrowdoc')
  server = Yarrow::Server.new
  server.run
end
