require 'awesome_print'
require 'json'
require 'net/http'
require 'maruku'
require 'yaml'

Dir[File.expand_path("lib/**/*.rb", File.dirname(__FILE__))].each{ |lib| require lib }

class FP < Thor
  desc :up, 'publish'

  CONFIG_FILE_NAME = File.join `echo $HOME`.strip, ".freepress.config"
  FRESH_SFW_PAGE_CREATOR = lambda { {'story' => []} }

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
        chunks = markdown.split(/\n{2,}/)
        sfw_page_data = FRESH_SFW_PAGE_CREATOR.call
        yaml = chunks.shift  # first chunk of Jekyll-formatted markdown is a YAML header: author, title, etc
        sfw_page_data['title'] = YAML::load(yaml)['title']

        # if the last markdown chunk is anchor tag definitions, eg:
        #   [BaseParadigm]: /BaseParadigm
        #   [Bitcask]: http://downloads.basho.com/papers/bitcask-intro.pdf
        # then store this chunk as link definitions, not as a markdown chunk
        link_defs = chunks.pop if chunks.last.match /(\[.+\]:\s+\S+\s*\n?)+/

        chunks.each do |chunk|
          chunk << "\n\n#{link_defs}" if link_defs
          sfw_page_data['story'] << ({
            'type' => 'paragraph',
            'id' => RandomId.generate,
            'text' => Maruku.new(chunk).to_html
          })
        end

        create_action = {
          'type' => 'create',
          'id' => RandomId.generate,
          'item' => sfw_page_data
        }
        create_action_json = JSON.pretty_generate(create_action)

        subdomain = "#{collection.split('.').first}.#{username}"
        page_file_name = File.dirname(page_path).split('/').last.parameterize
        action_path = "/page/#{page_file_name}/action"

        sfw_servers.each do |sfw_server|
          server, port = sfw_server.split(':')
          port = ( port || 80 ).to_i
          host = "#{subdomain}.#{server}"
          #puts create_action_json
          #puts "-- PUT #{host}:#{port}#{action_path}"
          connection = Net::HTTP.new(host, port)
          request = Net::HTTP::Put.new( action_path )
          request.set_form_data 'action' => create_action_json
          response = connection.request(request)
          raise( response.inspect ) unless response.code == '200'

          # TODO switch to rest-client gem:
          # require 'rest_client'
          # RestClient.put "http://example.com/resource", { 'x' => 1 }.to_json, :content_type => :json, :accept => :json
        end

      end
    end
  end
end

