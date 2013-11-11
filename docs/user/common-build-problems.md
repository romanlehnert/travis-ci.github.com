---
title: Common Build Problems
layout: en
permalink: common-build-problems/
---

<div id="toc"></div>

## Ruby: RSpec returns 0 even though the build failed

In some scenarios, when running `rake rspec` or even rspec directly, the command
returns 0 even though the build failed. This is commonly due to some RubyGem
overwriting the `at_exit` handler of another RubyGem, in this case RSpec's.

The workaround is to install this `at_exit` handler in your code, as pointed out
in [this article](http://www.davekonopka.com/2013/rspec-exit-code.html).

    if defined?(RUBY_ENGINE) && RUBY_ENGINE == "ruby" && RUBY_VERSION >= "1.9"
      module Kernel
        alias :__at_exit :at_exit
        def at_exit(&block)
          __at_exit do
            exit_status = $!.status if $!.is_a?(SystemExit)
            block.call
            exit exit_status if exit_status
          end
        end
      end
    end

## Ruby: Installing the `debugger_ruby-core-source` library fails

This Ruby library unfortunately has a history of breaking with even patchlevel
releases of Ruby. It's commonly a dependency of libraries like linecache or
other Ruby debugging libraries.

We recommend moving these libraries to a separate group in your Gemfile and then
to install RubyGems on Travis CI without this group. As these libraries are only
useful for local development, you'll even gain a speedup during the installation
process of your build.

    # Gemfile
    group :debug do
      gem 'debugger'
      gem 'debugger-linecache'
      gem 'rblineprof'
    end

    # .travis.yml
    bundler_args: --without development debug


## Ruby: rspec aborts without any reason

Some of us speed up their tests by disabling the garbage collection as mentioned 
[here](https://ariejan.net/2011/09/24/rspec-speed-up-by-tweaking-ruby-garbage-collection/).
Travis machines have 3GB ram and and when you take a look at the memory consumption 
of the tests without GC, you will notice that they reach that limit very fast. On travis
this will lead to problems. 

To have the GC disabled by default in the specs, but enabld on travis, make it dependent of an 
environment variable in spec/spec_helper.rb

    unless !!ENV['GC_ENABLED']
      puts "Garbage collection will be disabled for rspec. The process will leak. Enable it by setting GC_ENABLED=true"
      config.before(:all) { DeferredGarbageCollection.start }
      config.after(:all) { DeferredGarbageCollection.reconsider }
    end

And enable it in .travis.yml

    env: GC_ENABLED=true


## System: Required language pack isn't installed

The Travis build environments currently have only the en_US language pack installed. 
If you get an error similar to : "Error: unsupported locale setting", then you may need
to install another language pack during your test run.

This can be done with the follow addition to your `.travis.yml`:

    before_install:
      - sudo apt-get --reinstall install -qq language-pack-en language-pack-de

The above addition will reinstall the en\_US language pack, as well as the de\_DE
language pack.
