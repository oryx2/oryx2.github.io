---
layout: post
title: Writing your first Vim plugin using Ruby bindings
syntax: true
comments: true
---
As anyone who knows me is aware - I love Vim. I've been using it for a number of years now and I find it to be the most efficient way to write and edit code. I've spent countless hours customizing my vimrc and finding useful plugins but I've never actually written one myself. Recently I decided to change that and I thought I'd share my experience here. 

One of the biggest problems I had initially was finding a decent tutorial or documentation so we'll be walking through writing a trivial (and I do mean trivial) Vim plugin using Ruby and then I'll point you to some resources I found helpful for development.

So, let's get started! For this tutorial we'll be building a plugin called "Jumpback.vim." All it will do is provide a command that jumps the user back to their most recently viewed tab.

The first thing you should do is pick a global variable name to represent your plugin. That way you can ensure that it only gets loaded one time. We'll perform this check at the beginning of our file. Here we've decided on "loaded_jumpback"

{% highlight vim %}
if exists("g:loaded_jumpback")
  finish
endif
{% endhighlight %}

Next we need to make sure that our particular version of Vim is compiled with Ruby support. This is the most likely test to fail as many versions do not ship with Ruby support built in.

{% highlight vim %}
if !has("ruby")
    echohl ErrorMsg
    echon "Sorry, Jumpback requires ruby support."
  finish
endif
{% endhighlight %}

With that out of the way - we can fairly safely assume that our plugin will be loaded successfully. At this point we should set our "loaded_jumpback" variable to ensure the plugin is only loaded once.

{% highlight vim %}
let g:loaded_jumpback = "true"
{% endhighlight %}

Remember: The "g:" denotes a global scope so this variable will be accessible anywhere. For this reason you should try to make the name fairly unique. 

With that administrative stuff out of the way we can move on to setting up our Ruby bindings. Here we will be setting up Vim functions and mapping them to Ruby functions. That way we can write most of our code in Ruby which has substantially nicer syntax than Vimscript.

For this plugin we will need 3 functions. The "Jump" function is what the user will ultimately be able to call. It does the work of switching the tab back. The "Startup" function is called when Vim launches and it stores the initial tab. Finally, the "TabChanged" function is called every time the user switches tabs. This will allow us to keep track of which tab the user was previously on as well as which tab the user is currently on.

Here is how we map those Vim functions to their corresponding Ruby functions:

{% highlight vim %}
function! TabChanged()
  :ruby tab_changed
endfunction

function! Jump()
  :ruby jump_tab
endfunction

function! Startup()
  :ruby startup
endfunction
{% endhighlight %}

The structure of the plugin is starting to take shape at this point - now its time for the meat.

Two of the three functions we set up will need to be called automatically - one when Vim starts and the other when the user switches tabs. Doing this automatically will require use of Vim's autocmd functionality. We'll set up an autocmd group and bind our Vim functions (which in turn will call our Ruby functions) to the appropriate events. (You can see a list of available events with :help autocmd-events )

{% highlight vim %}
augroup autojumper
  autocmd TabEnter * :call TabChanged()
  autocmd GUIEnter * :call Startup()
augroup END
{% endhighlight %}

Just two steps remain in our little plugin! Lets set up a Vim command so that the end user can easily call it or bind it to the key of their choosing. 

{% highlight vim %}command Jumpback :call Jump(){% endhighlight %}

Now the user will be able to use our functionality by typing :Jumpback in their editor window. At this point we have everything but the Ruby code in place. There are several ways to include Ruby code in your vim file but we'll use the most straightforward. We just need to implement functions that match up to our earlier bindings and provide the desired functionality. The implementation is trivial - so I won't bother to explain it here but if you do happen to have any questions feel free to ask in the comments.

{% highlight vim %}
def tab_changed
  @previous = @current if @current
  @current =  VIM::evaluate("tabpagenr()")
end

def jump_tab
 if @previous
   VIM::command("tabn #{@previous}")
 end
end

def startup
  @current = VIM::evaluate("tabpagenr()")
end

EOF
{% endhighlight %}

And thats it! We're done! If you drop this file in your plugin directory and call :Jumpback it will work as desired. Now obviously this isn't a particularly useful plugin but it should show you the basics of how to set up and implement one. I'll leave you with some resources that I found helpful while learning myself.

[IBM Guide to Vimscript](http://www.ibm.com/developerworks/linux/library/l-vim-script-1/index.html) - Note that there are 5 parts, you can adjust the URL as needed but there don't appear to be links.

[Vim Ruby Bindings Docs](http://vimdoc.sourceforge.net/htmldoc/if_ruby.html)

[Jumpback.vim Source Code](https://github.com/thecoffman/Jumpback.vim)

I hope you found this helpful, if you have any questions feel free to leave them in the comments.
