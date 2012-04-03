# FreePress
#
# usage: bundle exec thor fp:up

require 'awesome_print'
require 'json'
require 'net/http'
require 'maruku'
require 'yaml'

Dir[File.expand_path("lib/ruby_extensions/*", File.dirname(__FILE__))].each{ |lib| require lib }

class FP < Thor
  desc :up, 'publish'

  CONFIG_FILE_NAME = File.join `echo $HOME`.strip, ".freepress.config"
  DEFAULT_SFW_PAGE = {'story' => []}

  def up
    config = YAML::load(IO.read(CONFIG_FILE_NAME))
    collection_path = config['collection_path']
    collections = config['collections']
    sfw_servers = config['targets']['smallest_federated_wiki']
    username = config['username']

    collections.each do |collection|
      page_pattern = File.join collection_path, collection, '*', 'index.md'
      page_pattern
      pages = Dir[page_pattern]
      pages.each do |page_path|
        markdown = IO.read(page_path).rstrip_lines!
        chunks = markdown.strip.split(/\n{2,}/)
        sfw_page_data = DEFAULT_SFW_PAGE.dup
        yaml = chunks.shift
        sfw_page_data['title'] = YAML::load(yaml)['title']

        # if the last markdown chunk is anchor tag definitions, eg:
        # [BaseParadigm]: /BaseParadigm
        # [Bitcask]: http://downloads.basho.com/papers/bitcask-intro.pdf
        # ...then store this chunk as link definitions, not as a markdown chunk
        link_defs = chunks.pop if chunks.last.match /(\[.+\]:\s+\S+\s*\n?)+/

        chunks.each do |chunk|
          chunk << "\n\n#{link_defs}" if link_defs
          sfw_page_data['story'] << ({
            'type' => 'paragraph',
            'id' => RandomId.generate,
            'text' => Maruku.new(chunk).to_html
          })
        end

        action = {
          'type' => 'create',
          'id' => RandomId.generate,
          'item' => sfw_page_data
        }

        subdomain = "#{collection.split('.').first}.#{username}"
        page_file_name = File.dirname(page_path).split('/').last.parameterize
        action_path = "/page/#{page_file_name}/action"

        sfw_servers.each do |sfw_server|
          server, port = sfw_server.split(':')
          port = ( port || 80 ).to_i
          host = "#{subdomain}.#{server}"
          connection = Net::HTTP.new(host, port)
          request = Net::HTTP::Put.new action_path
          request.set_form_data 'action' => action.to_json # JSON.pretty_generate(action)
          response = connection.request(request)
          raise( response.inspect ) unless response.code == '200'
        end

      end
    end
  end
end

module RandomId
  def self.generate
    (0..15).collect { (rand*16).to_i.to_s(16) }.join
  end
end