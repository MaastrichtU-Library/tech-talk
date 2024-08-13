This Jekyll blog theme is based on [The-MVM](https://github.com/the-mvm/the-mvm.github.io) (which on its turn is based on [Adam-blog](https://github.com/artemsheludko/adam-blog) and further adapted for the UB Maastricht use case.


## Run in production
The theme fully compliant with [GitHub pages](https://pages.github.com/) and can be deployed using GitHub actions.

## Run locally

### Preparation
Install the required packages as described in the [Jekyll docs](https://jekyllrb.com/docs/). For convenience, these are the steps in Ubuntu:

Install Ruby
```
sudo apt-get install ruby-full build-essential zlib1g-dev
```

Aset up a gem installation directory for your user account
```
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Install Jekyll and Bundler
```
gem install jekyll bundler

```

### Install dependencies
Install the site's dependencies from the Gemfile

Change directory to the folder where this repo and the `Gemfile` is located.
```
cd /path/to/this/workdir
```

Install Gems from Gemfile with
```
bundle install
```


### Run site
Start the site with
```
bundle exec jekyll serve
```

Access the site at http://localhost:4000



## More details
More details about Jekyll, the folder structure and workings can be found in the [MVM readme](https://github.com/the-mvm/the-mvm.github.io/blob/main/README.md).
