---
layout: post
title: Popping Threadsafe Containers
---
After spending a bit of time last week dealing with thread safety issues and a queue in C++, it only seems appropriate to blog a bit about it.  (Obviously, the information below would also apply to a stack or probably any other container type.)

Suppose you have a very common multiprocessing scenario:  two threads, one of which is producing work to be done, and another which is consuming the work to be done and performing the work.  You can accomplish this in a pretty straightforward fashion with:

* A queue
* Mutexes to prevent concurrent queue access
* A condition variable to tell the consumer when there is work to do

Originally I thought about creating my own container class for this.  Then part of my brain resisted.  “Quiet, engineering part of Matt’s brain!” it exclaimed.  “You’re just trying to do that because it seems Interesting!”  So I shut down that idea and pushed on trying to use a standard STL queue.  However, after I had to deal with some concurrency issues over and over, I realized that, at least this time, the engineering part of my brain was right.

So I set out to create a threadsafe queue.  The idea was that it would feel a lot like an STL queue, and would implement thread safety under the covers for you – so, for example, a simple call to q.empty() would automatically lock the mutex for you, see if the underlying queue was empty, then unlock and return the result.  So, essentially an STL queue wrapper class with automatic mutex locking, right?  It should be easy!

Well, you’d think so, until you get to an implementation of pop().

To see why this matters, first we should establish what we expect out of a threadsafe queue in C++:

* It should feel like an STL queue.  This means that it should have similar semantics and API if possible, and also, it should be a template.
* You get out what you put in.  Not a handle to what you put in; exactly what you put in is what you get out.
* Thread safety should be handled automatically – the user shouldn’t have to deal with thread safety.

An STL queue “pop()” isn’t as simple as it sounds.  The concept of “pop” is actually performed like this:

{% highlight c++ %}
    std::queue q;
    ...
    if (! q.empty()) {
        int v = q.front();
        q.pop();
    }
{% endhighlight %}

Popping an empty queue has strange consequences; in fact, on my machine, it segfaults.  You’d think the queue would be resilient to that behavior but it isn’t.  Also, you have to get the item off the front in a separate operation from actually removing the item.  This is so you can store anything in the queue – pointers, reference objects, etc. – and obtain the item in the front of the queue and know it will survive long enough for you to do something with it.  You pop() the queue after you are finished with the item.

That’s all fine but it makes our threadsafe queue a bit of a problem.  Suppose we decide to follow strictly the STL queue API.  In that case, we’d perform an “if not q.empty(), then q.front() and q.pop()” dance.  We have to assume that each of those three actions is legal on its own though, and we surely can’t release the lock after each one.  So that would mean that the user has to obtain a lock on their own, to be used outside of the whole three-step dance, which breaks one of our rules, namely, that the user shouldn’t have to deal with thread safety.

Okay, so what if we abandon the need for strict adherence to the STL queue semantics, and instead just implement a pop() method that does everything?  Now we can hide the thread safety management from the user.  But now we have a different problem.  How do we know whether the value we get back from pop() is a legal value?  How do we know whether it came from the queue or whether the queue was empty and there was nothing to pop()?  We need some way to check to see whether the value coming back is valid, i.e., whether there was something in the queue to pop beforehand.

There’s a couple of ways to do this, but none I like.

* If the return value were a pointer (or an iterator), we could check for NULL (or end()).  However, we don’t necessarily know that.  If the class is a template, and if we always get back what we put in, we can’t know whether the return value is a pointer or a value or a reference.  True, we could cast the return value to a pointer type – but then, not only are we not returning what we put in, but we are getting into a rather scary situation where the user is required to massage the data before it can be used properly.  And we lose type safety.
* We could return a ```std::pair<bool, t>``` where t is the type used to populate the queue.  Not as scary as the casting solution before, but still with the problem where the user has to massage the data, and the return value isn’t strictly what was put in (although it is contained in the pair).
* We could throw an exception if the container is empty, but an empty container isn’t exactly an exceptional case and is probably not a good ethical candidate for throwing an exception.
* We could stop using the STL queue and create our own queue with more reasonable semantics.  However, although I’m not necessarily in love with the STL queue, I’m also not convinced I can do it better, at least not convinced enough that I want to spend the time to do something that already exists.
* We could do away with making the wrapper class a template and instead make it implementation-specific.

Since I don’t know for sure that I will need my threadsafe queue for anything other than a very specific data type I have in mind right now, I chose the last option.  This is the agile way – I’ll meet today’s requirements today, and trust that I can refactor the code to meet future requirements when they show up.

So now everything works great, except doing the wait on the condition variable, which normally looks something like this:

{% highlight c++ %}
    mutex m;
    condvar cv;
    m.lock();
    if (q.empty()) cv.wait(m);
    v = q.pop();
{% endhighlight %}

Yeah, here I am, managing thread safety outside of the queue again.  I really should put this into my threadsafe queue also; except I would need a method something like ```waitThenDo(action)``` where ```action``` is a function pointer or some other callable.  I hate how that makes my code look though.  And at this point, with the waiting done as shown above (roughly), my code is working, so I’ve stopped caring.

But if you have a great solution to this problem, I’d love to hear it.