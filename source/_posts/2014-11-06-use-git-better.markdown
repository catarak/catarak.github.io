---
layout: post
title: "Use Git Better (and pay attention in lecture better)"
date: 2014-11-06 20:16:14 -0500
comments: true
categories: [Flatiron School, Git]
---

When I first learned git, I thought I understood it all. I had a grasp of the process of cloning a repository, making your own changes, and then making a pull request. I had made sense of it by making analogies to the old days when I worked in subversion. But then, I started to think about how to use git in production. And the question dawned on me: how do I synchronize with the master branch of the repository that I forked _from_? I understood the workflow of having a lot of local repositories and one remote, but when in came to multiple remote repositories, I was lost. 

As it turns out, it's easy to set up. First, make sure you've forked a repository, and have cloned your version locally. 

``` bash
$ git clone git@github.com:catarak/ruby-lectures-ruby-006.git && cd ruby-lectures-ruby-006
```

Then, if you're in the local directory of your cloned repository,

``` bash
[13:11:09] (master) ruby-lectures-ruby-006
$ 
```

You can see your configured remote repositories by running,

``` bash
$ git remote -v
origin  git@github.com:catarak/ruby-lectures-ruby-006.git (fetch)
origin  git@github.com:catarak/ruby-lectures-ruby-006.git (push)
```

Then add the remote repository that you forked from as "upstream," like so,

``` bash
$ git remote add upstream git@github.com:flatiron-school-ironboard/ruby-lectures-ruby-006.git
```

Then, if you run git remote -v again,

``` bash
$ git remote -v
origin  git@github.com:catarak/ruby-lectures-ruby-006.git (fetch)
origin  git@github.com:catarak/ruby-lectures-ruby-006.git (push)
upstream  git@github.com:flatiron-school-ironboard/ruby-lectures-ruby-006.git (fetch)
upstream  git@github.com:flatiron-school-ironboard/ruby-lectures-ruby-006.git (push)
```

You will see that you have added a new remote.

Now that we've set that up, you can now easily synchronize your fork. First, fetch the remote repository,

``` bash
$ git fetch upstream
remote: Counting objects: 22, done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 22 (delta 16), reused 19 (delta 13)
Unpacking objects: 100% (22/22), done.
From github.com:flatiron-school-ironboard/ruby-lectures-ruby-006
   7bfa768..68d09cc  master     -> upstream/master
```

Then, make sure you're on the master branch,

``` bash
$ git checkout master
```

And then merge in the changes. You may have some merge conflicts (if, say, you've been taking some lecture notes),

``` bash
$ git merge upstream/master
Auto-merging rails-lecture-7/todoapp/db/schema.rb
CONFLICT (content): Merge conflict in rails-lecture-7/todoapp/db/schema.rb
Auto-merging rails-lecture-7/todoapp/config/routes.rb
Auto-merging rails-lecture-7/todoapp/config/initializers/omniauth.rb
CONFLICT (add/add): Merge conflict in rails-lecture-7/todoapp/config/initializers/omniauth.rb
Auto-merging rails-lecture-7/todoapp/app/views/lists/index.html.erb
CONFLICT (content): Merge conflict in rails-lecture-7/todoapp/app/views/lists/index.html.erb
Auto-merging rails-lecture-7/todoapp/app/models/user.rb
CONFLICT (content): Merge conflict in rails-lecture-7/todoapp/app/models/user.rb
Auto-merging rails-lecture-7/todoapp/app/controllers/sessions_controller.rb
CONFLICT (content): Merge conflict in rails-lecture-7/todoapp/app/controllers/sessions_controller.rb
Auto-merging rails-lecture-7/todoapp/app/controllers/application_controller.rb
Auto-merging rails-lecture-7/todoapp/Gemfile.lock
CONFLICT (content): Merge conflict in rails-lecture-7/todoapp/Gemfile.lock
Auto-merging rails-lecture-7/todoapp/Gemfile
Automatic merge failed; fix conflicts and then commit the result.
```

If there are no conflicts, the changes will be commited automatically. If there are merge conflicts, add your changes and commit then:

``` bash
$ git add .
$ git commit -am "fixin merge conflicts"
[master b2c9367] fixin merge conflicts
```

And now, you're done.  

{% img center http://media.giphy.com/media/Y1N6D0KQODwzu/giphy.gif %}

The benefit of setting this up is that you can easily merge in the latest changes from the master of the project, but also have your own version. You can also not worry about accidentally pushing to the master of the repository you forked from.  

The benefit of this for me, currently, is that I'm a student at The Flatiron School. Our lecture notes are a repository. By doing this, I can take better notes in lecture. I like to try to keep up, and type everything that's happening in lecture. If I find that I have a typo or some other issue, I'll forget about it and keep writing stuff along with the instructor. Then, when lecture is over, I'll merge in what was happening in lecture, fixing everything that wasn't working during lecture. I find that by doing this, I'm more engaged in lecture and the information sticks. 
