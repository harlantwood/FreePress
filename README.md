# FreePress

Uploads collections of pages written in markdown to Smallest Federated Wiki instance(s).
Compatible with Jekyll YAML headers.

*Status*: Experimental!  Not fully baked. 

*Known issues*: Will only upload pages once.  If a page by that name already exists, the script will crash.

## Set up config file

    cp freepress.config.sample ~/.freepress.config

Edit ~/.freepress.config to reflect the locations of your markdown collections


## Run it

    bundle exec thor fp:up

No news is good news.

