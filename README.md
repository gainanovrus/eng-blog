# My Jekyll blog

## Preparation

### Setup system components

On Fedora 30
```
sudo dnf install ruby ruby-devel rubygems-devel \
                 autoconf automake bison gcc-c++ libffi-devel libtool \
                 libyaml-devel readline-devel sqlite-devel zlib-devel \
                 openssl-devel redhat-rpm-config rubygem-nokogiri

sudo gem install jekyll bundler
```

### View and test result

Deploy a site on localhost
```
bundler install
bundler update --bundler
bundle exec jekyll serve -d public --incremental --verbose --watch
```

Open deployment in browser ([localhost:4000/eng-blog](http:/localhost:4000/eng-blog))

## Deploy to GitHub

<!-- ### Prepare environment -->

<!-- Install npm
```
curl -sL https://rpm.nodesource.com/setup_10.x | sudo -E bash -
sudo yum install -y nodejs
``` -->

<!--
Install yarm
```
curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
sudo yum install -y yarn
``` -->

### Setup needed components

```
./bin/setup
```

### Deploy to gh-pages

Build project and deploy html-pages to `gh-pages` branch
```
./bin/deploy
```

## Open and test

URL: [gainanov.pro/eng-blog](https://gainanov.pro/eng-blog)
