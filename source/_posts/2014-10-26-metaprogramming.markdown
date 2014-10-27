---
layout: post
title: "Metaprogramming, in Brief"
date: 2014-10-26 23:03:36 -0400
comments: true
categories: [Flatiron School, Ruby, Metaprogramming]
---

Coming from a background in compiled languages, the idea of metaprogramming is fascinating to me. In the world of C, writting and running programs are distinct actions. At runtime, a program cannot be altered. However, in Ruby, the actual code can be modified while it is executing. To those who only know Ruby or similar interpreted languages, it may not seem like a big deal, but to me, I'm freaking out a little bit. I went to a hackthon once in which a guy wrote a self-modifying program and Python, and I thought it was black magic, but I didn't realize it was just a gift that comes with the language, and you didn't have to do any serious hacking to dynamically alter the code. 

Let's just look at eval. Maybe this isn't so exciting:

{% codeblock lang:ruby %}
eval("2 + 2") # => 4
{% endcodeblock %}

But then, consider this ditty:

{% codeblock lang:ruby %}
puts "Please enter a phrase: "
# pretend you entered Hello World here
phrase = gets.strip
puts "Enter the name of a string method: "
# pretend that you entered upcase here
method_name = gets.strip
exp = "'#{phrase}'.#{method_name}"
puts eval(exp) # => HELLO WORLD
puts "#{exp}" # => 'Hello World'.upcase
{% endcodeblock %}

That's pretty cool. You just wrote a little program on the fly. What else can you do? You can write a program that writes programs.

{% img center http://i.imgur.com/pz0sYGz.jpg %}

Yeah dawg. I'm for real. This is all of the code it takes (Adapted from [The Book of Ruby by Huw Collingbourne](http://www.sapphiresteel.com/The-Book-Of-Ruby)):

{% codeblock lang:ruby %}
program = ""
input = ""
line = ""
while line.strip() != "q"
  print( "?> " )
  line = gets.strip
  case line
  when ''
    puts "Evaluating..." 
    eval input 
    program += input 
    input = ""
  when 'l' 
    puts "Listing the program you've written so far..." 
    puts program 
  else
    input += line
  end 
end
{% endcodeblock %}

If you run this, you kind of have a REPL, but it's significantly dumber than IRB:

{% codeblock %}
?> def scramble(s)
?> s.split("").shuffle.join("")
?> end
?> 
Evaluating...
?> puts scramble("Hello")
?> 
Evaluating...
Holle
{% endcodeblock %}

Cool. So what other applications does metaprogramming have in Ruby besides writing REPL's? As it turns out, it's one of my favorite things I've learned about in Ruby thus far: ActiveRecord. When you define associations between models, whether it be belongs_to or has_many or anything else, the methods that you get for free are all generated on the fly. It does this using eigen or singleton classes. This is having the ability to define methods for specific instances of a class, after the class has already been defined. For example, say you define a class like so:

{% codeblock lang:ruby %}
class Roomba
  # Some other methods
  def vacuum_floor
    while floor.dirty?
      #vacuum for 5 seconds
      vacuum(5000)
      if clogged
        raise Roomba::CloggedException
      end
    end
  end
end
{% endcodeblock %}

Then you would instantiate it as

{% codeblock lang:ruby %}
fido = Roomba.new
{% endcodeblock %}

But say you got the newer model, which knows how to unclog itself. Then you could do something like

{% codeblock lang:ruby %}
def fido.vacuum_floor
  #vacuum for 5 seconds
  vacuum(5000)
  if clogged
    reverse_suction
  end 
end
{% endcodeblock %}

There's also this other fancy syntax you can use:

{% codeblock lang:ruby %}
class << fido
  def vacuum_with_style
    vacuum(5000)
    robo_boogie
  end
end
{% endcodeblock %}

It looks mysterious, but it's doing the same thing, I swear. 

You can actually define singleton methods for classes as well, and they're simply called class methods. You can do this in a number of ways:

{% codeblock lang:ruby %}
class Roomba
  def self.warranty
    "Lifetime"
  end
end
{% endcodeblock %}

This is the syntax we're familiar with. But there's more:

{% codeblock lang:ruby %}
class Roomba
  class << self
    def warranty
      "Lifetime"
    end
  end
end

def Roomba.warranty
  "Lifetime"
end

class << Roomba
  def warranty
    "Lifetime"
  end
end
{% endcodeblock %}

ActiveRecord creates new singleton methods for instances and classes when you define associations. It may seem like magic, but that magic is just metaprogramming.

{% img center http://media.giphy.com/media/ujUdrdpX7Ok5W/giphy.gif %}