source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem "tzinfo-data", platforms: [:mingw, :mswin, :x64_mingw, :jruby]

# Performance-booster for watching directories on Windows.
# 0.1.x cannot compile on Ruby 3.x (missing rb_thread_call_without_gvl), so 0.2+ is required.
# Declared with `platforms:` rather than `if Gem.win_platform?` so the Gemfile evaluates
# identically on Windows and on CI's Linux runner, keeping any Gemfile.lock portable.
gem "wdm", "~> 0.2", platforms: [:mingw, :mswin, :x64_mingw]
