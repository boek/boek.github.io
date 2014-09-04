---
layout: post
title:  "Syntax highlighting for toto"
date:   2012-08-13 14:44:09
categories:
---

toto is great, but with a development oriented blog I needed a clean way to display code to my readers.
So first thing on my todo list was to add Code block support with Syntax highlighting to Toto.

to turn this:
<pre>
    def do_something(test)
      test.each do |t|
        puts t
      end
    end
</pre>
into this:

{% highlight ruby %}
def do_something(test)
  test.each do |t|
    puts t
  end
end
{% endhighlight %}

After [some](https://www.ruby-toolbox.com/categories/syntax_highlighting) [research](http://railscasts.com/episodes/207-syntax-highlighting?view=asciicast   ) I settled on a gem called [coderay](http://coderay.rubychan.de/).
You can install it using `gem install coderay`

Next I did some _[Monkeypatching](http://www.codinghorror.com/blog/2008/07/monkeypatching-for-humans.html)_.
{% highlight ruby %}
require 'coderay'

#open the Toto module
module Toto
  class Article
    #overrided body and summary to use coderay
    def body
      markdown coderay(self[:body].sub(@config[:summary][:delim], '')) rescue markdown self[:body]
    end

    def summary length = nil
      config = @config[:summary]
      sum = if self[:body] =~ config[:delim]
        self[:body].split(config[:delim]).first
      else
        self[:body].match(/(.{1,#{length || config[:length] || config[:max]}}.*?)(\n|\Z)/m).to_s
      end
      markdown(coderay(sum.length == self[:body].length ? sum : sum.strip.sub(/\.\Z/, '&hellip;')))     
    end

    #adding a method for coderay
    def coderay(text)
      text.gsub(/\<code( lang="(.+?)")?\>(.+?)\<\/code\>/m) do
        CodeRay.scan($3, $2).div(:css => :class, :line_numbers => :table)
      end
    end
  end
end
{% endhighlight %}

After that I added `require './toto-extend.rb'` to `config.ru` to load our file


And then I styled it using my favorite theme [Solarized](http://ethanschoonover.com/solarized). You can grab the css [here](/css/coderay.css).

Still have a few kinks to work out, but this should get you going. If you need any help feel free to give me a shout on [Twitter](http://twitter.com/jeffboek)

***

#Edit

Not surprising there is a *better* way to add functionality to a gem. Instead of Monkeypatching it in, you can Fork the repo on Github. Make your changes, and reference it in your Gemfile like so


`gem "toto", :git => "git@github.com:jeffboek/toto.git"`