source "https://rubygems.org"

gem "jekyll", "~> 3.9"

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.13"
  gem "jekyll-seo-tag", "~> 2.6"
  gem "jekyll-sitemap", "~> 1.4"
  gem "jekyll-remote-theme", "~> 0.4"
  gem "jekyll-paginate", "~> 1.1"
  gem "jemoji", "~> 0.11"
  gem "kramdown", ">= 2.3.0"
  gem "kramdown-parser-gfm", "~> 1.1"
end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :install_if => Gem.win_platform?

