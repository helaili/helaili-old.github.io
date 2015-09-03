---
layout: post
title:  "What's this Git fast forwarding stuff?"
date:   2015-03-16 10:00:00
categories: Git
comments: True
---

I have recently decided to go quite deeper into Git and GitHub. I have been a GitHub user since 2012, but my usage was quite basic and I was always committing against my <code>master</code> branch. Reading about branching and merging, I came across the fast-forward concept which gave me a bit of headache so I decided to experiment to visualize the differences between a fast-foward and a non fast-forward commit, and as a result figure out what is a merge commit.

First step is to create a <code>ff-blog-entry</code> branch from my <code>gh-pages</code> which is my publication branch for this blog. I then commit some changes, merge, push it to GitHub and delete the branch.

{% highlight shell-session %}
git checkout gh-pages
git checkout -b ff-blog-entry
git add .
git commit -m "New blog entry about fast forward"
git checkout gh-pages
git merge ff-blog-entry
git push origin gh-pages
git branch -d ff-blog-entry
{% endhighlight %}

As you can see in the screenshot below, this merge resulted in a new dot on the <code>gh-pages</code> commit timeline. There is no reference to the fact we used a branch named <code>ff-blog-entry</code> to work on the content.

![Commit flow with fast forward]({{ site.url }}/assets/fast_forward_merge.png)

Now let's do something slightly different and add the <code>--no-ff</code> to the merge command. Note that a commit message is required when using this flag.

{% highlight shell-session %}
git checkout -b ff-blog-entry
git add .
git commit -m "Adding more content for the non ff merge"
git checkout gh-pages
git merge --no-ff -m "Non Fast Forward merge of content" ff-blog-entry
git push origin gh-pages
git branch -d ff-blog-entry
{% endhighlight %}

The outcome is quite different from the fast-forward merge. We now clearly see the commit happened in a different branch which was later merged with <code>gh-pages</code>.

![Commit flow without fast forward]({{ site.url }}/assets/non_fast_forward_merge.png)

And while the first commit/merge operation created one single commit in our branch history, the latter one introduced two commits : the content related commit, and the merge commit. 

![Commit history]({{ site.url }}/assets/commit_history.png)

Fast forwarding is nice in the sense it allows to simplify the commit history, but it's not always an option. If others have made conflicting changes, then merging is mandatory, no fast forwarding possible here. But you still might want to show a slick and easy-to-read history. The magic of rebasing comes here... well, in an other post rather. 
