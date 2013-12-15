---
title: "Make a shoplist app with D3"
layout: default
lang: en
---

# A shoplist with D3


<div class="text-center" style="float:right;width:260px;margin-left:50px">
    <iframe src="http://bl.ocks.org/lud/raw/7869565/" width="250" height="550" style="border:none"> </iframe>
</div>

 insert introduction text here ^^

<div class="clearfix"></div>


## Data

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
classes. CSS classes cannot contain spaces
* `s` is a numeric identifier. The list items do not store the
list name but this numeric value, which is more
small/efficient. 's' stands for status (todo, done, ...)
* `wf` is a list of the workflow targets i.e. the 'n' of the
other lists an item from this list can be moved to.
* `label` is a label ... a display text
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

## Manipulate the data

Now we need some tools to access data in a convenient manner.

We define a function to refenrence a spec from its `n` or it's `s` :

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
of a list. The current implementation works if we send a list `n` but
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

There is a function to move all items from one list to another. Here
we start from the end because we are implementation-aware of how items
are stored and moved. This is not good. But this example should be
simple. We won't define a Class-like constructor for the lists.

{% highlight javascript %}
    function moveAll(from,to) {
        var listFrom = getList(from);
        var i = listFrom.length;
        Moving the items once at a time. Good enough ?
        while(i--) {
            listFrom[i][to]();
        }
    }
{% endhighlight %}

Finally, for each item in our allItems lists, we create an Item
object. Since in it's constructor, an Item add itself to the right
list, there is nothing more to handle here.

{% highlight javascript %}
    allItems.map(function(spec){
        new Item(spec);
    });
{% endhighlight %}

Our data is now ready. Just `console.log(lists);`.

## Display the data with D3

We will use our lists specs as data to display the lists headers and
create lists items containers (`<ul/>`). Then, for each list, we will
append our list items (`<li/>`) to theese containers.

First, we select our wrapper and then do a data join with `.selectAll`
and `.data`. We supply the listspecs as data. We will call this
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

Then we append to theese groups the header, a div element with the
list icon, the list label (or the list `n` if the label is falsy (like
an empty list)), and an empty span which will display the number of
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

It's time to display our list items. But we will use the lists specs
again to update the list-len display. Remeber, the data-join between
elements whith `.div-group` CSS class and `listspecs` is still valid.
We can inspect the corresponding list-spec for each group, and so get
the associated list and its length.

Let's simply call `render()`. At each time we call this function, the
DOM is updated to reflect the current state of the data. The function
definition is just next.

{% highlight javascript %}
    render();

    function render() {
{% endhighlight %}

First, we update the item count on each list header

{% highlight javascript %}
        group.selectAll('.list-len')
            .text(function(ls){ return ' ('+getList(ls).length+')' });
        group.each(function(ls){
{% endhighlight %}

The D3 function `.each()` allow to work with only a group at a time.
it provides us the possibility to use closures.

we make a data-join on our `<li/>` elements and our items. The key are
item's `n` property

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
related to items but this is not mandatory. If the list is not empty,
we select any possible paragraph existing and remove it.

We must do this after we have removed the old tags which were moved

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
items that doesn't change persist in the DOM. On the first render, all
items are new

So, on our new items, let's create a `<li/>` element. We append a
`<span/>` element whose text will be the item name (`n` property).

{% highlight javascript %}
            var newli = li.enter().append('li');
            newli.append('span')
                .text(function(d,x){ return d.n });
{% endhighlight %}


Then, we want to be able to move an item from a list to another, but
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

We have added to the Item prototype functions such as `resupply`,
`later`, etc. There we iterate on the `wf` property of the current
list spec, which contain the same names. So, for each other list an
item can be sent to, we just register a listener for the `click` event
which call this function on the Item instance.

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

