---
layout: post
title:  "Imagining atomatizing the App"
date:   2014-12-15 21:35:43
categories: om clojurescript
---

This morning, while reflecting on a late night of working through David Nolen's (swannodette) Intro to Clojurescript and [Om][om] [Basic Tutorial][om-tut], I had an epiphany, an insight into a solution to a problem I've been working on for about 15 years.  The goal I've been seeking is an elegant app for a non-programmer user to create a custom, elegant scientific data collection and visualization app.  It requires building up a data survey from simple and complex data types, possibly based on an existing denormalized database schema, and linking that with an easily customizable data entry screen.  It's kind of like Access, but more complex while also easier to use.  Yes, a paradox.  XRX was a step in the right direction, but using XML to define and query data sucks.  ODK is doing amazing things with it, but only by reverting to HTML and javascript for the UI.

I've taken a few runs at the solution over the years, starting with a naive PHP app in the foggy beginning of my programming career, and a couple of paid database gigs for clients for similar needs, but I have yet to attain the degree of Nirvana I know is possible.  When I became familiar with Clojure, I intuited that it would probably be much more useful for the job, due to its homoiconic nature, being represented in its own data format and amenable to meta-programming, and to its immutable data types - a severely useful toolbox.  But the language has been opaque to me, and simultaneously both too complex and too simple to see how to use it to its fullest ... until this morning.  Suddenly, while looking through my bay windows to the trees behind my monitor, I saw the architecture I needed to use.  Kind of a mix between a spreadsheet tree, data schema UML, Pure Data wiring, a Swift playground, etc, but that changes based on position within the multidimensial data abstraction context tree.

One possible step toward making this app easier to reason is the definition of the app itself, which Om's component-based nature largely enables.  However, I'm wondering if there's an additional step that could be taken to a simpler declarative definition of the app, which would be easier for a user or another app to construct and edit.  For example, here's a prototypical Om component definition:

{% highlight clojure %}
(defn comment [{:keys [author text]} owner]
  (om/component
    (dom/div #js {:className "comment"}
      (dom/h2 nil author)
      (dom/span nil text))))
{% endhighlight %}
[source][fredyr-gist]

It's a very nice declarative way to define a component, and reminds me of Enyo's nested declarative javascript component model:

{% highlight javascript %}
{kind: "onyx.Button", name:"Image Button", ontap:"buttonTapped", components: [
  {tag: "img", attributes: {src: "assets/enyo-logo-small.png"}},
  {content: "There is an image here"}
]}
{% endhighlight %}
[source][enyo]

Enyo is the closest tool I've found yet to enabling the workflow I need.  However, it feels funky.  I know, I'm picky.

Clojure's Lispiness is very amenable to representing the enclosing tag nature of XML and HTML, and is preferrable if the UI is being generated programmatically anyway.  In that vein, Enyo's definition is more clear in some ways because it is simpler, relying on a convention or schema for being converted into running code.  But what if the Om approach were to be similarly abstracted further?  Can the following simpler version of (fredyr)'s component work?

{% highlight clojure %}
{ :name :comment
  :class "comment"
  ;:kind :div ; :div is the default
  :content [
   { :kind :h2, :state :author}
   { :kind :span, :state :text}]}
{% endhighlight %}

It is easier for me to read and seems to contain all of the same essential info, without a lot of the repetitive code.  This declaration is taking advantage of some conventions.  One is that a component will pass to its children the same state that it received, unless otherwise stated (pun intended).  It's unnecesary to declare its input variables, and it only defines more refined sub-cursors when handing state to its children.  This declaration could be used to generate the CLJS code above.  This component uses no state of its own; it's just a container DIV.  This is another convention: a component is by default a DIV, unless otherwise noted.  Here's another more complex component, first in Om code:

{% highlight clojure %}
(defn comment-list [cs owner]
  (om/component
    (dom/div nil
      (dom/div #js {:className "commentList"}
        ;; Helper function build-all builds a vector of components
        (om/build-all comment cs))
      (dom/button #js
        {:onClick #(change-author cs "Fredrik")}
        "Change authors"))))
{% endhighlight %}

and here its mapped abstraction:

{% highlight clojure %}
{ :name :commentList
  ; Declares that this component accepts a vector of :comment components.  A Schema might be nice, but really, it's all convention over configuration, a la Enyo, yes?
  :state [:comment]
  :content [
    { :class "commentList"
      :content [
        {:kind :comment, :loop :all}]}
    { :kind :button
      :onClick [:changeAuthor :state "Frederik"]
      :content "Change authors"}]}

{% endhighlight %}

I'm unsure about the mapping of the "build-all" macro, but I'm confident there's some reasonable way to represent it.  My impression is that the mapped form of the component delarations would be much easier for me to reason around with the aim of building an app that could read and write this structure.

Plus, as if that weren't enough, there's a huge bonus to this approach: undo and change branching (and saving, and sharing) FOR FREE!  Non-programmers are overwhelmed enough trying to make a relatively sophistaced app, but this would make it much easier to save different versions, try something for a while, save it, go back a few steps, go in a different direction, save it, and then be able to browse around through the different versions.  This might even make it possible to more easily diff different versions.



[fredyr-gist]:      https://gist.github.com/fredyr/8460923
[enyo]:   https://github.com/enyojs/onyx/blob/master/samples/ButtonSample.js
[om]: https://github.com/swannodette/om
[om-tut]: https://github.com/swannodette/om/wiki/Basic-Tutorial
