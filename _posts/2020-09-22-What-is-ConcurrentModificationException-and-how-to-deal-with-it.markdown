---
layout: post
title: "What is ConcurrentModificationException and how to deal with it?"
date: 2020-09-22 13:32:20 +0300
description: Generally, it is not permissible to iterate over a collection and perform modification on it at the same time. But with iterator, we can find a work around.
img:  java-cme.png
tags: [Java] # add tag
---
In my recent development work of implementing LDAP user synchronization, one of the features is to check if there are any inactive users so then to remove them from LDAP server. A code snippet looks like this:
{% highlight java %}
    List<String> users = new LinkedList<String>(Arrays.asList("Alice", "Bob", "Carol"));
    for (String name : users) {
        users.remove("Alice");
    }
{% endhighlight %}

Obviously, this will throw an `ConcurrentModificationException`. But why? Let's take a look at the `Collection` implementation:
{% highlight java %}
   public void remove() {
       if (lastRet < 0)
           throw new IllegalStateException();
       checkForComodification();

       try {
           ArrayList.this.remove(lastRet);
           cursor = lastRet;
           lastRet = -1;
           expectedModCount = modCount;
       } catch (IndexOutOfBoundsException ex) {
           throw new ConcurrentModificationException();
       }
   }

   final void checkForComodification() {
       if (modCount != expectedModCount)
           throw new ConcurrentModificationException();
   }
{% endhighlight %}

As in Javadoc, it states that the iterators returned by this class's {@link #iterator() iterator} and methods are <em>fail-fast</em>: if the list is structurally modified at any time after the iterator is created, in any way except through the iterator's own {@link ListIterator#remove() remove} or {@link ListIterator#add(Object) add} methods, the iterator will throw a {@link ConcurrentModificationException}.  Thus, in the face of concurrent modification, the iterator fails quickly and cleanly, rather than risking arbitrary, non-deterministic behavior at an undetermined time in the future.


You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:



Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
