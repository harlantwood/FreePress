require 'net/http'
require 'yaml'

class SDP < Thor
    desc :up, 'publish'

    CONFIG_FILE_NAME = "selfdotpublish.yml"
    DEFAULT_SFW_PAGE = { 'title' => name, 'story' => [] }

    def up
        p config = YAML::load(IO.read(CONFIG_FILE_NAME))
        p 111, collections = config['collections']
        p 222, collection_path = config['collection_path']
        collections.each do |collection|
            page_pattern = File.join(collection_path, collection, '*', 'index.md')
            pages = Dir[page_pattern]
            pages.each do |page|       # /Users/harlan/
                markdown = IO.read(page)
                chunks = markdown.split(/(\s*\n){2,}/)
                sfw_page_data = DEFAULT_SFW_PAGE.dup
                sfw_page_data[:title] = YAML.read(chunks.shift)[:title]
                chunks.each do |chunk|
                    sfw_page_data['story'] << ({
                        'type' => 'paragraph',
                        'id' => RandomId.generate,
                        'text' => md2html(chunk)
                    })
                end
                action = {
                    'type' => 'create', 
                    'id'   => RandomId.generate, 
                    'item' => {'title' => page}
                }
                
                server = "harlan-knight.#{collection.split('.')[0]}.sfw.remixfreeip.org"
                uri = URI("http://#{server}/page/#{page}/action")
                res = Net::HTTP.post_form(uri, :action => JSON.pretty_generate(sfw_page_data))
                
                # put "/page/#{page}/action", :action => JSON.pretty_generate(sfw_page_data)"
            end
        end
    end
end

# selfdotpublish.yml
# collections_path: /Users/harlan/HKB/Nodes
# collections: 
#       - enlightenedstructure.org
#       - The_Project
# 
# targets
#   - smallest_federated_wiki
#       - sfw.remixfreeip.org
# 

module RandomId
    def self.generate
        (0..15).collect { (rand*16).to_i.to_s(16) }.join
    end
end