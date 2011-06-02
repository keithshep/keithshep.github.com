---
layout: post
title: "F#: building async data structures on top of MailboxProcessor"
date: 2011-06-01
---

I'm just diving into using async workflows in F# and they're great. If you're not familliar with what async workflows are and how they work I recommend:

* the "Multithreaded and Concurrency Applications" section of the [F# Programming](http://en.wikibooks.org/wiki/F_Sharp_Programming) wiki book
* Microsoft's documentation on [Asynchronous Workflows](http://msdn.microsoft.com/en-us/library/dd233250.aspx) and the [Control Namespace](http://msdn.microsoft.com/en-us/library/ee340440.aspx)

[MailboxProcessor](http://msdn.microsoft.com/en-us/library/ee370357.aspx) allows asynchronous tasks to communicate with eachother but one thing you'll notice when you read the doc is that it "... encapsulates a message queue that supports multiple-writers and a single reader agent", which doesn't quite fit my current need. I need for my tasks to be able to communicate through a message queue that supports many readers and many writers with a guarantee that a message will only be read once.

## Wrapping MailboxProcessor to allow for many readers

We can implement a concurrent data structure on top of MailboxProcessor that allows many readers and many writers (here I implemented a stack because delivery order and fairness doesn't matter for my use case):

{% highlight ocaml %}
type StackMessage<'Item> =
    | PushMsg of 'Item
    | PopMsg of AsyncReplyChannel<'Item>
    | PopAllAndQuitMsg of AsyncReplyChannel<'Item array>

type AsyncStack<'Item> () =
    let goAgent (inbox : MailboxProcessor<StackMessage<'Item>>) =
        let rec processNextMsg (stack : 'Item list) (deferredPops : AsyncReplyChannel<'Item> list) =
            async {
                let! nextMsg = inbox.Receive ()
                match (nextMsg : StackMessage<'Item>) with
                | PushMsg item ->
                    match deferredPops with
                    | [] -> return! processNextMsg (item :: stack) deferredPops
                    | popHead :: popTail ->
                        popHead.Reply item
                        return! processNextMsg stack popTail
                | PopMsg popReplyChan ->
                    match stack with
                    | [] -> return! processNextMsg stack (popReplyChan :: deferredPops)
                    | headItem :: stackTail ->
                        popReplyChan.Reply headItem
                        return! processNextMsg stackTail deferredPops
                | PopAllAndQuitMsg popReplyChan ->
                    popReplyChan.Reply (Array.ofList stack)
                    return ()
            }
        
        processNextMsg [] []
    
    let agent = MailboxProcessor.Start goAgent

    member this.Push (item : 'Item) = agent.Post (PushMsg item)
    member this.Pop () = agent.PostAndAsyncReply (fun replyChan -> PopMsg replyChan)
    member this.PopAllAndQuit () =
        agent.PostAndAsyncReply (fun replyChan -> PopAllAndQuitMsg replyChan)
{% endhighlight %}

## What about BlockingCollection?

One of the nice things about F# is that we can directly use the full .Net library in our F# code (even if some of it isn't so elegant when used in functional code). I think you could just as well use [BlockingCollection](http://msdn.microsoft.com/en-us/library/dd267312.aspx) to solve this problem but if you do be very careful not to block your async tasks since that will cause the threadpool to explode if you have many readers.

