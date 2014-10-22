---
layout: post
title: "6 Things about Ruby Regular Expressions That Will Change Your Life Forever"
date: 2014-10-13 12:03:23 -0400
comments: true
categories: [Ruby, Regular Expressions, Flatiron School]
---

### 1. The Equals Tilde
What's an equalstilde, you say? Who would come up with such a dumb sounding name for an operator? It actually lies at the foundation of Ruby regular expressions. It allows you to apply a regular expression to a string, and returns the index within the string where the regular expression matches. For example, 

{% codeblock lang:ruby %}
/Lisa/ =~ "You're tearing me apart, Lisa!" #=> 25
{% endcodeblock %}

### 2. Matchdata
Instead of using the equalstilde, you can also use a string method called match to apply a regular expression to a string. However, instead of returning an index, it returns this weird type of object called MatchData:

{% codeblock lang:ruby %}
"You're tearing me apart, Lisa!".match(/Lisa/) #=> #<MatchData "Lisa"> 
{% endcodeblock %}

Cool, so then what do you do with a MatchData object? It all makes sense when you learn about this crazy thing called...

### 3. Capture groups
Get ready to have your mind blown. In Ruby, if you use parenthesis in a regular expression, you can utilize capture groups. You can extract multiple parts from a string without using multiple regular expressions, just by putting the part of the string you want to capture. Get out, I know. It's awesome. For example,

{% codeblock lang:ruby %}
m = """I mean, the candles, the music, the sexy dress... 
       what's going on here?"""
m.match(/the (.*), the (.*), the (.*)\.\.\./)
# => #<MatchData "the candles, the music, the sexy dress..." 
# 1:"candles" 2:"music" 3:"sexy dress"> 
{% endcodeblock %}

Then, you can access each of the capture groups separately, like so:

{% codeblock lang:ruby %}
m[0] # => "the candles, the music, the sexy dress..."
m[1] # => "candles" 
m[2] # => "music"
m[3] # => "sexy dress"
{% endcodeblock %}

You can even name your capture groups:

{% codeblock lang:ruby %}
m = """I mean, the candles, the music, the sexy dress... 
       what's going on here?"""
m.match(/the (?<sexy_item_1>.*), the (?<sexy_item_2>.*), the (?<sexy_item_3>.*)\.\.\./)
# => #<MatchData "the candles, the music, the sexy dress..." 
# 1:"candles" 2:"music" 3:"sexy dress"> 
{% endcodeblock %}

Then you can access each group using hash syntax:

{% codeblock lang:ruby %}
m[sexy_item_1] # => "candles"
{% endcodeblock %} 

### 4. Atomic Grouping

An atomic group is a type of capture group. When the regex engine exits it, all backtracking positions are discarded. Let's go over two cases, one that uses atomic grouping, and one that doesn't, and see how the regex engine would operate. 

{% codeblock lang:ruby %}
"Tommy Wiseau".match(/\A(Tommy|Thomas|Tom)\z/) # => nil 
{% endcodeblock %}

The regex engine first matches the start of the string, \A, and then matches "Tommy". However, since it then would leave the capture group and try to match the \z, or the end of a string, the match would fail. The engine would then go back and try to match Thomas, and fail, try to match Tom, and ultimately stop and declare failure. But say we want to shorten this process. 

{% codeblock lang:ruby %}
"Tommy Wiseau".match(/\A(?>Tommy|Thomas|Tom)\z/) # => nil
{% endcodeblock %}

In this case, again, the \A is matched as the start of the string, and then the engine tries to match "Tommy". It succeeds and moves onto matching \z, which fails. Because of the atomic grouping, the engine has thrown out all back tracing data upon reaching the \z, and therefore fails after only trying to match "Tommy" rather than all three options in the capture group.

### 5. Subexpression Calls

Okay. Okay. This one is probably the most radical thing about regular expressions.

By using the \g<name> syntax, you can match a previously named subexpression, which can be a group name or number. An example better demonstrates how you use it.  

Say that you want to make sure all parenthesis surrounding a string are always balanced. You would use something like this:

{% codeblock lang:ruby %}
"(spoons)".match(/(?<paren>\([^()]*\g<paren>*\))/) 
# => #<MatchData "(spoons)" paren:"(spoons)"> 

"((spoons))".match(/(?<paren>\([^()]*\g<paren>*\))/) 
# => #<MatchData "((spoons))" paren:"((spoons))">

"((spoons)".match(/(?<paren>\([^()]*\g<paren>*\))/) 
# => #<MatchData "" paren:nil>
{% endcodeblock %}

Cool. Pretty cool. Let's go over what the regex engine does.

1. Enter a capture group named *paren*.
2. Match a literal *(*.
3. Match the text in between the parenthesis that is anything except for parenthesis.
4. Call the *paren* capture group again, dropping the part in the middle of the parentheses for now.
5. Enter the *paren* capture group again
6. Match a literal *(*, the second character in the group
7. Match the text in between the parenthesis that is anything except for parenthesis.
8. Try to call *paren* again, but fail since it would cause the match thus far to fail.
9. Match a literal *)* n times, where n is the depth of the recursion. 

Note that the * following \g<paren> is extremely important. This allows the regular expression to break out of recursion, since the subexpression can exist 0 or many times. Without it, the recursion would be never ending. 

### 6. Lookahead and lookbehind assertions

What if you want to make sure certain characters exist in a regular expression, but you don't want them to be part of your match group? This is when you would want to use a special type of anchor, called lookahead and lookbeind assertions.

* *(?=pat)* is a positive lookahead assertion, and ensures that the characters following your expression match "pat"
* *(?!pat)* is a negative lookahead assertion, and ensures that the characters following your expression do not match "pat"
* *(?<=pat)* is a positive lookbehind assertion, and ensures that the characters preceeding your expression match "pat"
* *(?<!pat)* is a negative lookbehind assertion, and ensures that the characters preceeding your expression do not match "pat"

Pretty fantastic, right? For example, say you have a list of emails, and you're trying to find the usernames of all of the ones at a certain domain:

{% codeblock lang:ruby %}
"mark@theroom.com".match(/.*(?=@theroom.com)/) 
# => #<MatchData "mark"> 

"johnny@theroom.com".match(/.*(?=theroom.com)/) 
# => #<MatchData "johnny"> 

"michael@troll2.com".match(/.*(?=theroom.com)/) 
# => nil
{% endcodeblock %}

## Other References

If there's still more you want to know about regular expressions in Ruby, I recommend looking at the [Ruby Docs](http://www.ruby-doc.org/core-2.1.3/Regexp.html) or visiting the webstite [Regular-Expressions.info](http://www.regular-expressions.info/), which contains more than you'd ever want to know about regular expressions. In the best way possible. 
