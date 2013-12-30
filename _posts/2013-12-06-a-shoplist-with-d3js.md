---
title: "Make a shoplist app with D3"
layout: default
lang: en
---

# A shoplist with D3

D3js is a fantastic tool to work with graphics in javascript. But here
I want to show that it's also excellent to do whatever you want as
long as you have **data**. Most of applications on the internet use
"views", as in MVC, or MVVM, etc. I bet that if you have a "view"
somewhere, it's worth to try D3.

To follow this tutorial, you must be able to read and write javascript
code and know how to use D3 data joins, mainly the `.selectAll(...)`,
`.data(...)`, `.enter()`, `.exit()`, etc. functions, and know what functions
passed to `.attr(...)` receive as parameters. My code here is really
not hard but I do not explain theese concepts. ([Some doc
here](http://bost.ocks.org/mike/selection/)).

My code is far from perfect, It would be glad to get pull requests
[here](https://gist.github.com/lud/7869565).

### What is it ?

<div class="text-center" style="float:right;width:260px;margin-left:50px">
    <iframe src="http://bl.ocks.org/lud/raw/7869565/" width="250" height="550" style="border:none"> </iframe>
    <a href="http://bl.ocks.org/lud/raw/7869565/" target="_blank">Open in a new window.</a>
</div>

This is a simple application I use to manage the goods I have in my
kitchen. At any time I know what I have at home, and what I need to
buy. So, if I am in a grocer's shop because I suddenly want to eat
some chocolate, I can consult this app on my smartphone and buy what I
need.

It consists of four lists of items, with buttons to move items from
one list to another.

The "Goods to resupply" is the lists of items I miss, those I want to
buy. It's the only list I have to consult on my smartphone so it's on
top of the page.

If I don't want to buy some item when I'm shopping, I can click on
<button class="btn btn-primary btn-xs"><span class="glyphicon
glyphicon-time"></span></button> to send the item to the `later` list.
It keeps the top list clean, without unneeded items.

When I put some food in my cart, I send the corresponding item to the
`incart` list with <button class="btn btn-primary btn-xs"><span
class="glyphicon glyphicon-shopping-cart"></span></button>. This is
the "check" button.

Then, the `incart` list shows what I have on my cart. If suddenly I
don't want a particular item no more, I send it back to the "Goods to
resupply" list with <button class="btn btn-primary btn-xs">   <span
class="glyphicon glyphicon-refresh"></span> </button>.

When I'm done shopping, I send all the items in my cart to the
`athome` list with <button class="btn btn-primary btn-xs">   <span
class="glyphicon glyphicon-home"></span> </button> and I know that I
have them. Except if my car burns in a car accident, but this
application does not support car accidents ...

I use a different web page to add or remove items from the
application. I do this at home on a computer with a wide screen, this
is not implemented yet. We are currently dicussing the mobile part of
the app only.

But I could add a button to create items, and a fifth list called
`trash` to send items to it and then remove them.

The application misses buttons on lists headers to move all the items
from a list to another ; notably from `later` to "Goods to resupply"
(Or from `athome` to the top, call it the car accident button). I did
implement them but I was not happy with the result. The code was very
short but the look and feel was not good.

<div class="clearfix"></div>

## How we make it ?

It's time to talk about code. We will see now how do we build this
simple list application with a relative small amount of code, thanks
to the expressiveness of both javascript and D3js.

This tutorial is just the real code with some comments between each
parts of it. You can see all of the HTML and javascript code
[here](http://bl.ocks.org/lud/7869565).

We do not talk about server side here, there is no AJAX code, but as
soon as an item is moved to one list, it should be updated on the
server. (Or put in a queue that send updates and retries when the shop
building is too thick for the waves !).

**Update:** I like short variable names, but it can be confusing for
some readers. I speak a lot about `n` and `s` properties. Think
about them in terms of `name` and `id`, respectively.

### Data

First we have some lists specifications. In a real world app, this
would come from the server.



{% highlight javascript %}
    var listspecs =
        [ {n:'resupply', s:1, wf:['later','incart'],    label:'Goods to resupply', icon:'refresh'       }
        , {n:'later',    s:2, wf:['incart'],            label:'',                  icon:'time'          }
        , {n:'incart',   s:3, wf:['resupply','athome'], label:'',                  icon:'shopping-cart' }
        , {n:'athome',   s:4, wf:['resupply'],          label:'',                  icon:'home'          }
        ];
{% endhighlight %}

Theese specifiactions are objects with the following members :

* `n` is the simple name of the list. It must be a valid unique
identifier because it's used as function names and CSS
classes. CSS classes cannot contain spaces.
* `s` is a numeric identifier. The list items do not store the list
name but this numeric value, which is more small/efficient. 's' stands
for status, the status of an item, it's current list.
* `wf` is a list of the workflow targets i.e. the `n` of the
other lists an item from this list can be moved to.
* `label` is a label ... a display text.
* `icon` is an identifier for an image. This could be a
filename or anything. Here we use Twitter Bootstrap and
it's Glyphicons. So this is a "glyphicon-" prefixed CSS
class name.

Then we have some items :


{% highlight javascript %}
    var allItems = [ {s: 1, n: 'Oranges'}
                   , {s: 1, n: 'Apples'}
                   , {s: 4, n: 'Carots'}
                   , {s: 2, n: 'Grapes'}
                   , {s: 2, n: 'Pears'}
                   , {s: 4, n: 'Bananas'}
                   , {s: 1, n: 'Potatoes'}
                   , {s: 1, n: 'Ice Cream'}
                   , {s: 1, n: 'a new Laptop'}
                   ];
{% endhighlight %}

`n` is the name of the item, and `s` shows in which lists the item is
when we load the page. This `s` property matches with the lists' `s`
property.

Finally, we want to display multiple lists of items. D3 likes data as
lists, so we will create as many lists (Arrays) as there are lists
specs. For the moment, let's declare a simple object to associate
lists names (resupply, later, ...) with actual Arrays.

{% highlight javascript %}
    var lists = {};
{% endhighlight %}

### Manipulate the data

Now we need some tools to access data in a convenient manner.

We define a function to reference a spec from its `n` or it's `s`.
`listspecs` is an array, if we want to retrieve a list specification
in this array, we must know the index of the element in it. For each
list spec, we associate it's `s` property with it's index, and it's
`n` property with it's index too, in a registry variable (`listReg`).
Now, we can retrieve a list specification index from it's `n` or `s`,
and acces the right entry in the array, in the defined function.

{% highlight javascript %}
    var getListSpec = (function(){
        var listReg = {};
        listspecs.forEach(function(ls,i){
            listReg[ls.s] = i;
            listReg[ls.n] = i;
        });
        return function(x) {
            return listspecs[listReg[x]];
        }
    }());
{% endhighlight %}

We define a function to get the actual list (Array object) with the
same arguments : `s` or `n`, or a listspec object.

{% highlight javascript %}
    function getList(x) {
        var n = x.n || getListSpec(x).n;
        return lists[n];
    }
{% endhighlight %}

Let's define a simple constructor (class-like) to add methods to our
items. An item can remove itself from a list (`degroup`), add itself
to a list (`addTo`) and keep a reference to it (`this.list`). The
first call to addTo is made within the constructor. addTo uses the `s`
of a list. The current implementation works if we send a list's `n` but
we may send to a sever our changes, so we should stick with one value-
type only.

{% highlight javascript %}
    var Item = function(itemSpec) {
        this.s = itemSpec.s;
        this.n = itemSpec.n;
        this.list = undefined;
        this.addTo(itemSpec.s);
    };
    Item.prototype.degroup = function(){
        if(this.list) {
            this.list.splice(this.list.indexOf(this),1);
        }
    };
    Item.prototype.addTo = function(s){
        var list = getList(s);
        this.list = list;
        list.push(this);
    };
{% endhighlight %}

Now, for each existing spec, we will create an entry in the `lists`
object, and add to the list a reference to it's spec. We will also
define methods to the Item prototype to allow us to send an item to a
specific list. After this code, we can call `(new
Item({...})).resupply();`.

{% highlight javascript %}
    listspecs.forEach(function(ls){
        lists[ls.n] = [];
        lists[ls.n].spec = ls;
        Item.prototype[ls.n] = function(){
            this.degroup();
            this.addTo(ls.s);
        };
    });
{% endhighlight %}

Finally, for each item in our `allItems` list, we create an `Item`
object. Since in it's constructor, an Item add itself to the right
list, there is nothing more to handle here.

{% highlight javascript %}
    allItems.map(function(spec){
        new Item(spec);
    });
{% endhighlight %}

Our data is now ready. Just `console.log(lists);`.

### Display the data with D3

We will use our lists specs as data to display the lists headers and
create lists items containers (`<ul/>`). Then, for each list, we will
append our list items (`<li/>`) to theese containers.

First, we select our wrapper and then do a data join with `.selectAll`
and `.data`. We supply the lists specifications as data. We will call this
elements a 'group' : a header and the items.

{% highlight javascript %}
    var group = d3.select('#lists')
        .selectAll('div.group')
        .data(listspecs);
{% endhighlight %}

We create the groups wrappers : the same elements that we
targeted in `selectAll()`.

{% highlight javascript %}
    var newgroup = group.enter().append('div').attr('class','group');
{% endhighlight %}

Then we append to theese groups the header, a `div` element with the
list icon, the list label (or the list `n` if the label is falsy (as
is an empty list)), and an empty span which will display the number of
items in the list.

{% highlight javascript %}
    var grouphead = newgroup.append('div').attr('class','grouphd');
    grouphead.append('span')
        .text('')
        .attr('class',function(ls){ return clicon(ls.icon) })
    grouphead.append('span')
        .text(function(ls){ return ls.label || ls.n })
        .attr('class', 'list-name')
    grouphead.append('span')
        .attr('class', 'list-len')
{% endhighlight %}

And then, a `<ul/>` tag with the lists `n` property as an id

{% highlight javascript %}
    newgroup.append('ul')
        .attr('id', function(ls){ return ls.n })
        .attr('class','list list-unstyled');
{% endhighlight %}

It's time to display our list items. We will define a function
`render`, and call it every time the data is updated (when an item is
moved). Our the data-join between `div.group` (elements) and
`listspecs` (data) exists forever (as long as we do not make a new
join on the same elements). As the data changes, we can update our
elements.


First, we update the item count on each list header

{% highlight javascript %}
    function render() {

        group.selectAll('.list-len')
            .text(function(ls){ return ' ('+getList(ls).length+')' });
        group.each(function(ls){
{% endhighlight %}

The D3 function `.each()` allow to work with only a group at a time.
it provides us the ability to use closures.

We make a data-join on our `<li/>` elements and our items. The key are
item's `n` property. When calling `group.each(function(){ ... })`,
inside the function, `this` refers to the current group's DOM element.

{% highlight javascript %}
            var items = getList(ls);

            var li = d3.select(this).select('ul').selectAll('li')
                .data(items, function(d) { return d.n });
{% endhighlight %}

We remove the old entries which aren't in the list no more

{% highlight javascript %}
            li.exit().remove();
{% endhighlight %}

If the current list is empty, we display a paragraph to inform the
user. We also stop execution of the function since all further code is
related to items but this is not mandatory.


If the list is not empty, we select any possible paragraph existing
and remove it. We do this before adding the paragraph on an empty
list.

{% highlight javascript %}
            d3.select(this).select('p.empty').remove();

            if (items.length === 0) {
                d3.select(this).append('p')
                    .attr('class','empty')
                    .text('Nothing here');
                return;
            }
{% endhighlight %}

Now we are only concerned with the new entries in the list, since the
items that doesn't change persist in the DOM. On the first rendering,
all items are new.

So, on our new items, let's create a `<li/>` element. We append a
`<span/>` element whose text will be the item name (`n` property).

{% highlight javascript %}
            var newli = li.enter().append('li');
            newli.append('span')
                .text(function(d,x){ return d.n });
{% endhighlight %}

At this point, we can see the items. If you don't need buttons, you're
done.

But we want to be able to move an item from a list to another, but
only following workflow rules. We will now append as much buttons as
there are target lists in the `wf` property of our current list spec.
We just append a wrapper for theese buttons and pass this new element
to a function thanks to the d3 function `.call()`.

{% highlight javascript %}
            newli.append('div')
                .attr('class','ctrl')
                .call(function (wrapper) {
{% endhighlight %}

The callback is sent the selection of all the wrapper for the current
list. This is a d3 selection, so we can use it as if it was a single
element `<div/>`. It is called only once per group.

We have added functions such as `resupply()` and `later()` to the
`Item.prototype`. There we iterate on the `wf` property of the
current list spec, which contain the same names. So, for each other
list an item can be sent to, we just register a listener for the
`click` event which call this function on the `Item` instance.

When we click on a button, the item is moved to another list, so data
changes. We need to call `render()` again to update our elements.

Finally, the button is empty, so we append a `span` with a `class`
attribute containing what Bootstrap needs to display a Glyphicon. See
the `clicon` function below.

{% highlight javascript %}
                    ls.wf.forEach(function(n){
                        wrapper.append('button')
                            .attr('class','btn btn-primary btn-xs')
                            .on('click',function(d){
                                d3.event.preventDefault();
                                d[n]();
                                render();
                            })
                            .append('span')
                                .text('')
                                .attr('class',clicon(getListSpec(n).icon))
                    });
                });
        })
    }
{% endhighlight %}

This is a helper function to transform an icon name into a glyphicon
class.

{% highlight javascript %}
    function clicon(i) {
        return 'glyphicon glyphicon-'+i;
    }
{% endhighlight %}


Now, let's simply call `render()` "manually" to trigger the first
display.

{% highlight javascript %}
    render();
{% endhighlight %}

And we're done ! Thanks for reading.
