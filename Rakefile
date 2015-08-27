require 'yarrow'
require 'yarrow/output/context'
require 'rom'

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
end

class CreatePost < ROM::Commands::Create[:memory]
  register_as :create
  relation :posts
  result :one
end

$rom = ROM.finalize.env

include Yarrow::Tools::FrontMatter

task :collect do
  Dir['content/posts/*.md'].each do |path|
    content, data = extract_split_content(File.read(path, :encoding => 'utf-8'))
    $rom.command(:posts).create.call(data.merge(body: content))
  end
end

task build: :collect do
  puts $rom.relation(:posts).to_a
end