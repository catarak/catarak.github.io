---
layout: post
title: "Use Git Better (and pay attention in lecture better)"
date: 2014-11-06 20:16:14 -0500
comments: true
categories: [Flatiron School, Git]
---

When I first learned git, I thought I understood it all. I had a grasp of the process of cloning a repository, making your own changes, and then making a pull request. It made analogies to the old days when I worked in subversion. But then, I started to think about how to use git in production. And the question dawned on me: how do I merge in changes from the master branch of he repository that I forked _from_? I understood working with a bunch of local repositories and one remote, but what if there are multiple remote repos? 

As it turns out, it's pretty easy to set up. If you're in your cloned repository,

{% img left /images/git_images/command_line Command Line %}

You can see your configured remote repositories by running,

{% img left /images/git_images/remote_before All Remotes Before %}

Then add the remote repository that you forked from as "upstream," like so,

{% img left /images/git_images/remote_add Add Remote %}

Then, if you run git remote -v again,

{% img left /images/git_images/remote_after All Remotes After %}

You will see that you have added a new remote.

Now that we've set that up, you can now easily synchronize your fork. First, fetch the remote repository:

{% img left /images/git_images/fetch_upstream Fetch Upstream Master %}

Then, merge in the changes. You may have some merge conflicts (if, say, you've been taking some lecture notes):

{% img left /images/git_images/merge_conflicts Fix Merge Conflicts %}

If there are no conflicts, the changes will be commited automatically. If there are merge conflicts, add your changes and commit then:

{% img left /images/git_images/git_commit %}

And now, you're done.  

The benefit of setting this up is that you can easily merge in changes from the master of the project, but also have your own version and not worry about accidentally pushing to master. 

The benefit of this for the students of The Flatiron School is that you can take better notes in lecture. I like to try to keep up, and type everything that's happening in lecture. If I find that I have a typo or some other issue, I'll forget about it and keep writing stuff along with the instructor. 

The prerequisite to this is that you've forked the Ruby Lectures repository, but I think all of you already know how to do that. 
