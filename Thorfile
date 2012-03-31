# FreePress
#
#

require 'json'
require 'net/http'
require 'maruku'
require 'yaml'

class FP < Thor
    desc :up, 'publish'

    CONFIG_FILE_NAME = File.join `echo $HOME`.strip, ".freepress.config"
    DEFAULT_SFW_PAGE = {'story' => []}

    def up
        config = YAML::load(IO.read(CONFIG_FILE_NAME))
        collection_path = config['collection_path']
        p 111, collections = config['collections']
        collections.each do |collection|
            page_pattern = File.join collection_path, collection, '*', 'index.md'
            p page_pattern
            p pages = Dir[page_pattern]
            pages.each do |page|
                markdown = IO.read(page)
                chunks = markdown.split(/(\s*\n){2,}/)
                sfw_page_data = DEFAULT_SFW_PAGE.dup
                p 333, yaml = chunks.shift
                sfw_page_data['title'] = YAML::load(yaml)['title']
                chunks.each do |chunk|
                    sfw_page_data['story'] << ({
                        'type' => 'paragraph',
                        'id' => RandomId.generate,
                        'text' => Maruku.new(chunk).to_html
                    })
                end
                action = {
                    'type' => 'create',
                    'id' => RandomId.generate,
                    'item' => sfw_page_data.merge({'title' => page})
                }

                subdomain = "harlan-knight.#{collection.split('.')[0]}"
                p 555, server = "#{subdomain}.sfw.remixfreeip.org"
                connection = Net::HTTP.new(server)
                p 666, action_path = "/page/#{page}/action"
                #request = Net::HTTP::Put.new action_path
                #request.set_form_data :action => JSON.pretty_generate(sfw_page_data)
                #response = connection.request(request)
                #p 444, response


                #connection = Net::HTTP.new("api.restsite.com")
                #request = Net::HTTP::Put.new("/users/1")
                #request.set_form_data({"users[login]" => "changed"})
                #response = connection.request(request)

            end
        end
    end
end

module RandomId
    def self.generate
        (0..15).collect { (rand*16).to_i.to_s(16) }.join
    end
end