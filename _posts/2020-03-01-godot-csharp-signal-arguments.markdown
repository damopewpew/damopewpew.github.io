---
layout: post
title: "Godot - C# Signals with Arguments"
date: 2020-02-29 13:23:00 +1030
categories: godot C#
---

I have been using Godot a bit lately with C#. It's going pretty well but I'm finding some of the documentation for certain things to be lacking on the C# side of things.

One case in particular was passing down data into a signal on connect and getting that data when the signal is fired. 

### Real life use case

I had a bunch of nodes that I instatiated in a loop and connected the on_Press signal to a function in the parent node. I wanted to be able to tell which object fired the event.

### Code Example

{% highlight csharp %}
using Godot;
using GC = Godot.Collections;
using System.Collections.Generic;

public class Parent : Spatial
{

    [Export]
    private PackedScene _child;

    private Array<PackedScene> children = new Array<PackedScene>();

    public override void _Ready()
    {
        this.Initialize(10);
    }

    public override void Initialize(uint amount) {
        for (uint i = 0; i < amount; i += 1) {
            // Create instance of the child
            PackedScene child = this._child.Instance();  

            children.push(child);

            var args = new GC.Array();

            // We add the index of our child to the arguments array
            args.Add(i);

            // Connect the signal and add our arguments array
            child.GetNode("RididBody").Connect("input_event", this, nameof(_on_Child_InputEvent), args);

            AddChild(child);
        }
    }

    /** Add the idx to the end of our arguments
     * We can add as many arguments as we like
     */
    public override void _on_Child_InputEvent(object camera, object @event, Vector3 click_position, Vector3 click_normal, int shape_idx, uint idx)
    {

        // Get our child by the index we passed in as a the argument
        var child = children[idx]; 

        // Do something with it
    }
}
{% endhighlight %}

I'm not sure if this is the correct way to handle this situation but it works well for my situation.

Hope this helps anyone who has come across the same issue and found no resources on how to fix it.
