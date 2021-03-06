It's straight up trivial.  Also, look into ZAngband - more content, less trying to be funny.

The phrase you're looking for is "roguelike development."  This turns out to be a long standing dev community (except for the javascript part.)

----

# Getting started

Anyway, suppose you have a map that looks like

    ##+#########
    #@....######
    #...R.##...#
    #.$...##.w.#
    ###+####...#
    ###....=..w#
    ########...#
    ############

Rendering that is easy a zillion ways.  Since you're new, I think you should use a `<table>`, because that's easy to wrap your head around; an experienced person would probably instead absolutely position `<span>`s inside a fixed-size scrollable container, but let's not bother with that for now.  Baby steps.

First, you need the map data as a value in your JavaScript.  For now we'll just use that fixed map; making your own map is one of the fun parts.

Note: \n means "new line."  It's like hitting return.

    var map = '##+#########' +  // todo: remove player, creatures, objects
            '\n#@....######' +
            '\n#...R.##...#' +
            '\n#.$...##.w.#' +
            '\n###+####...#' +
            '\n###....=..w#' +
            '\n########...#' +
            '\n############';

A good design wouldn't use the letters to store the map state, and neither will we, but this gets us a place to start.  I often proceed by putting bullshit in place, then writing notes reminding me what to remove, then removing that as I go; it lets me work in a more "complete" space, where things are (fake) already in place, which I find easier.

If you do this, *don't* skip the reminder comments.

----

# Having a page to draw into

Next, let's draw that in our document.  We'll need a host document (`index.html`,) which might look something like this:

    <!doctype html>
    <html>

      <head>
        <script defer type="text/javascript" src="rogue_demo.js"></script>
        <link rel="stylesheet" type="text/css" href="rogue_demo.css"/>
        <title>Rogue demo</title>
      </head>

      <body></body>

    </html>

At this point, please put a simple `window.alert('hello');` in the relevant javascript file, and set the body background color to orange in the relevant CSS file, and load the result.  This ensures that all files are in appropriate places, that the tags have the filenames and attributes correct, et cetera.

Assuming those both go well, remove the alert and the orange, and let's please continue.

----

# The worst looking render

A simple way to get some satisfaction immediately - and don't underestimate how important that is - is to throw that map on screen, so that you know you're getting something.

Let's do that.  Please put the map variable from above in, and then what we'll do is write two functions - `MapDraw` and `onStart`.

## MapDraw

The first function, `MapDraw`, we'll start in idiot-simple territory.  We're going to create an HTML element `<pre>`, which makes fixed-width non-scrolling non-stripped text, and we're going to fill it with the map text, then return that.

    function MapDraw(map) {
      return '<pre>' + map.toString() + '</pre>';
    }

Later we'll replace this with a much better one.

## onStart

This is sort of our checkered flag to go.  Web browsers will emit an event called `window.onload` when everything's ready to begin.  What we need to do is create a function (we'll call it an "event handler", which doesn't have a technical meaning in this language, but rather reminds us of its purpose) whose goal is to hook `window.event` and do our setup tasks.  At this time that only means drawing the map, though that will be a growing list.

    function onStart() {
      document.body.innerHTML = MapDraw(map);
    }

And we also need to assign that to `window` as the function meant to handle that event, like so:

    window.onload = onStart;

## And that's a soup

Your end result should look something like this:

    var map = '##+#########' +  // todo: remove player, creatures, objects
            '\n#@....######' +
            '\n#...R.##...#' +
            '\n#.$...##.w.#' +
            '\n###+####...#' +
            '\n###....=..w#' +
            '\n########...#' +
            '\n############';

    function MapDraw(map) {
      return '<pre>' + map.toString() + '</pre>';
    }

    function onStart() {
      document.body.innerHTML = MapDraw(map);
    }

    window.onload = onStart;

Save that and the HTML in some directory (the HTML source expects the JS to be called `rogue_demo.js`,) and open the index; you should now see a shitty, fully JS map being rendered.

[Step 1](todo whargarbl)

----

# An Aside

> Why not just keep it in a `<pre>` forever, then?

Because having individually placed cells is easier to hook events (click, hover, touch, etc) to, and for spells, that needs to include empty floor and walls, so that's everything, not just items and monsters, so wrapping everything in `<span>`s quickly becomes silly.

.

> But I heard that `<table>` was bad

Uh huh.

.

> Why not just place `<span>`s directly?

Oh, you can.  That's actually a good route.  But it involves math, bounding boxes, forced empty rendering, and some hairy portability things, and this is a noob, so I'm trying to keep it simple.  Tables are simple.

.

> Flexbox?

Shut up.

----

# It looks weird

I know.  It's the web.  Let's start fixing that.  Make a CSS file called `rogue_demo.css` and let's put a starting step towards it.

    body, html, pre { margin: 0; padding: 0; border: 0; }

That's CSS-ese for "don't put any spacing or lines around these three tags."  Save that, reload, and watch it look slightly less terrible.

We're going to start putting effort into the appearance later.  For now, it's good enough to see that it's under control.  Effort expended now would be wasted; we're about to change how that map is drawn.

----

# Less-bad map

There are a tremendous number of ways to represent a map, and some of the most interesting roguelike design can be found in fundamental map representation.  However, that's a difficult topic.  Today we'll do a straightforward map.

There are two approaches to a straightforward map.  Each has their upsides and downsides.  You see the other above: a string representation.  That's a good approach, actually, for a lot of things, particularly for hand-drawn special regions.  However, it also has a whole *lot* of problems, like that it makes procedural generation, your bread and butter, borderline impossible.

The reason it's such a great first step, though, is actually very compelling: you take a look at it and you know exactly what you're looking at.  High class roguelikes (the Angband, Crawl, Hack, ADOM, Torchlight, and Diablo families, for example) have a whole lot of fixed special areas; a few of them (FTL, X-Com, Syndicate) are made entirely or nearly entirely out of fixed special areas.  These are very powerful tools, and you want to be able to make them quickly, easily, and while having fun, yeah?

Let's start by showing the other approach; after that, I can explain how we'll handle the dichotomy.  (Pro tip: we're gonna cheat.)

## Map-as-a-grid

The closest thing we have to a grid in JavaScript is an array-of-arrays, unless you're going off into the woods and implementing some shit in classes or decorators or whatever, which, y'know, don't.

Why this approach is better is simple: you get to refer to cells like map[3][5], which means drawing a box here and a line there and a random cloud over yonder is pretty straightforward, because numbers.  If you were trying to do that on a string, you'd have to do a bunch of math to figure out where in the string it went (and by the way, javascript strings aren't iteration-fast, because of unicode technicalities, so that'll be slooooow.)

So that looks something like (don't worry, I'll de-horrible this afterwards):

    var map = [ [ '#', '#', '+', '#', '#', '#', '#', '#', '#', '#', '#', '#' ], // todo: remove player, creatures, objects
                [ '#', '@', '.', '.', '.', '.', '#', '#', '#', '#', '#', '#' ],
                [ '#', '.', '.', '.', 'R', '.', '#', '#', '.', '.', '.', '#' ],
                [ '#', '.', '$', '.', '.', '.', '#', '#', '.', 'w', '.', '#' ],
                [ '#', '#', '#', '+', '#', '#', '#', '#', '.', '.', '.', '#' ],
                [ '#', '#', '#', '.', '.', '.', '.', '=', '.', '.', 'w', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '.', '.', '.', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
              ];

Now, obviously you don't want to write maps like that by hand.  But, already some improvements are obvious: the newlines are gone, the math to see what's north or south is just add one or subtract one, it's straightforward to blot out regions, et cetera.

Later, we'll go back and resurrect map strings, because they matter a lot.  But for now, let's get this new one working.

## Now we render this new map as a `<table>`

Since we have to gut and redo the map draw, this is as good a time as any to switch to `<table>`, too.  It's actually easier for this map type anyway.

Let's write a few simple arrow functions to handle this for us.  An arrow function is an anonymous function that takes either a bare argument or a list of arguments, then implicitly returns the statement you issue.  If there's only one argument you can skip the `()` framing on the arglist, and if there's only one statement you can skip the `{}` framing on the body.  This leads to extremely terse lambdas.

Check it out:

    var RenderCell =    cell => '<td>' + cell.toString() + '</td>',
        RenderRow  =     row => '<tr>' + (row.map(RenderCell).join('')) + '</tr>',
        RenderMap  = mapdata => '<table><tbody>' + (mapdata.map(RenderRow).join('')) + '</tbody></table>';

That says
  * RenderCell is an arrow function, taking cell, yielding the cell to string framed in a `<td>`
  * RenderRow is an arrow function, taking a row, which maps the row as an array with `RenderCell`, joins the result with nothing inbetween, and frames it in a `<tr>`
  * RenderMap is an arrow function, taking a mapdata, which maps the mapdata as an array with `RenderRow`, joins the result with nothing inbetween, and frames it in `<table><tbody>`.

This work is all currently done with strings.  That won't stay true forever, but it works for now.

At this point the new map renders, but is no longer in a fixed-width font; on most systems the `@` is much wider than the other characters, and so the map will appear to be slightly broken.  That's okay; this is a progress point `:)`

We also update our `onStart` to call `RenderMap` instead of `MapDraw`.

The new JS should look something like this:

    var map = [ [ '#', '#', '+', '#', '#', '#', '#', '#', '#', '#', '#', '#' ], // todo: remove player, creatures, objects
                [ '#', '@', '.', '.', '.', '.', '#', '#', '#', '#', '#', '#' ],
                [ '#', '.', '.', '.', 'R', '.', '#', '#', '.', '.', '.', '#' ],
                [ '#', '.', '$', '.', '.', '.', '#', '#', '.', 'w', '.', '#' ],
                [ '#', '#', '#', '+', '#', '#', '#', '#', '.', '.', '.', '#' ],
                [ '#', '#', '#', '.', '.', '.', '.', '=', '.', '.', 'w', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '.', '.', '.', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
              ];

    var RenderCell =    cell => '<td>' + cell.toString() + '</td>',
        RenderRow  =     row => '<tr>' + (row.map(RenderCell).join('')) + '</tr>',
        RenderMap  = mapdata => '<table><tbody>' + (mapdata.map(RenderRow).join('')) + '</tbody></table>';


    function onStart() {
      document.body.innerHTML = RenderMap(map);
    }

    window.onload = onStart;

And now our map is rendered as a `<table>`, which has important benefits coming up.

Except &hellip; we've lost string maps `:(`

[Step 2](todo whargarbl)

----

# The string map conundrum

The thing is, hand drawn regions are important for making a game setting.  ***Really important***.  So, we want them, even though the above array map is super obviously better.

## The cheating part

The key thing we need to do is to make a thing that takes one and makes the other.  The benefits of the string one are to the human, and the benefits of the array one are to the machine and to your software, so, let's take the string one, and make the array one.  's actually pretty easy.

There are more efficient ways to do this which are less easy.  I'm going to use ES6; this won't work in older browsers, and setting up shims or `babel.js` is outside the scope of this discussion.  `dealwithit.jpg`

    var mapStringToGameMap = mapString => mapString.split('\n').map( row => row.split('') );

What that does:

1. Takes a `mapString`, which we intend to convert into a `GameMap`, by
1. taking the `mapString`,
1. `split`ing it on the newline, yielding an array of each row,
1. `map`ping (sorry, coincidence) over those rows with
1. the `arrow function` "`row => row.split('')"
1. which means the end result is that arrow function's result over each row
1. the arrow function just says "take the row and return the result of `split()`ing it"
1. in javascript, split() with an empty string argument on a string yields an array of each letter
1. which got done to each row (by the `map()`,) as a way to build our result
1. which is our new `GameMap`, an array of arrays of letters
1. and return that result

Later, as GameMaps become more powerful, that function will have to, too.  (Or rather, we'll make a class after all, and use this as one way to initialize them.)

But for now that will do.

## Wossit do

Let's hook it up.  Let's grab the old string version of the map; let's change the `w`s in the sample map to `p`s so you can see the difference between the two maps visually; then let's hook up string maps in our array map system.

First, the old string map, slightly altered:

    var str_map = '##+#########' +  // todo: remove player, creatures, objects
                '\n#@....######' +
                '\n#...R.##...#' +
                '\n#.$...##.p.#' +
                '\n###+####...#' +
                '\n###....=..p#' +
                '\n########...#' +
                '\n############';

Next, using it as a source is straightforward; we just wire onStart to call for different initial data:

    function onStart() {
      document.body.innerHTML = RenderMap(mapStringToGameMap(str_map));
    }

Your source should now look something like this:

    var str_map = '##+#########' +  // todo: remove player, creatures, objects
                '\n#@....######' +
                '\n#...R.##...#' +
                '\n#.$...##.p.#' +
                '\n###+####...#' +
                '\n###....=..p#' +
                '\n########...#' +
                '\n############';

    var map = [ [ '#', '#', '+', '#', '#', '#', '#', '#', '#', '#', '#', '#' ], // todo: remove player, creatures, objects
                [ '#', '@', '.', '.', '.', '.', '#', '#', '#', '#', '#', '#' ],
                [ '#', '.', '.', '.', 'R', '.', '#', '#', '.', '.', '.', '#' ],
                [ '#', '.', '$', '.', '.', '.', '#', '#', '.', 'w', '.', '#' ],
                [ '#', '#', '#', '+', '#', '#', '#', '#', '.', '.', '.', '#' ],
                [ '#', '#', '#', '.', '.', '.', '.', '=', '.', '.', 'w', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '.', '.', '.', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
                [ '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#', '#' ],
              ];

    var RenderCell =              cell => '<td>' + cell.toString() + '</td>',
        RenderRow  =               row => '<tr>' + (row.map(RenderCell).join('')) + '</tr>',
        RenderMap  =           mapdata => '<table><tbody>' + (mapdata.map(RenderRow).join('')) + '</tbody></table>',
        mapStringToGameMap = mapString => mapString.split('\n').map( row => row.split('') );

    function onStart() {
      document.body.innerHTML = RenderMap(mapStringToGameMap(str_map));
    }

    window.onload = onStart;

And now we have an array map system rendering from map strings; the best of both worlds `:)`

Let's do something with that world.  But first, another de-uglying pass.

[Step 3](todo whargarbl)

----

First, in the `RenderMap` arrow function, let's give the `<table>` an `id` that we can hook rules to shamelessly.

    RenderMap = mapdata => '<table id="gamemap"><tbody>' + (mapdata.map(RenderRow).join('')) + '</tbody></table>',

Next, in the CSS, let's start by enforcing an actual square grid.  Let's also make the page background slightly gray, and the default map cell white text on black background, which is obviously wrong.  More on that soon.

    body        { background-color: #eee; }
    #gamemap    { border-collapse: collapse; }
    #gamemap td { height: 1em; width: 1em; background-color: black; color: white; }

Let's also add some rules to the map cells that prevent overflow, that strip out default extra spacing and border lines, and that center text both horizontally and vertically within the cells (the last piece won't work outside table cells.)

    #gamemap td { height: 1em; width: 1em; background-color: black; color: white; overflow: hidden;
                  margin: 0; padding: 0; border: 0; text-align: center; vertical-align: middle; }

Now things look dramatically less awful.  Let's get some active map behavior in, so that we can get properly started.

Fortunately we can start making faster steps now, too.

[Step 4](todo whargarbl)

----

# Let's interpret the map

So right now we're having the map fly blind on strings.  That's awful.  It's a nice bootstrapping notation but it won't do properly.  Let's have it be interpreted into something reasonable, instead; then we can get a better map to be rendered.

We need two pieces to safely undo that; afterwards we can do whatever dumb thing we like.  First, we need to make a cell object, instead of just some single letter string; second, we need to modify the table builder to call the ostensible method on the cell class which tells it what to contain.

Honestly I'd like to do the cell object as an ES6 class, but your browser probably doesn't support them by default yet, and I don't want to cover `babel` this morning.

## Making a map cell

First, the `cell` is (initially) simple enough: take an input, bind it to a closure variable, return an object that craps out the closure variable on request.  Cool.

    function cell(input) {
      var content = input;
      return {
        htmlRepresentation: function() { return content; }
      };
    }

[Step 6](todo whargarbl)

# Using the map cell

It's time to start actually leveraging our map.

Next we change the string map consumer to apply the individual characters as the cell argument, instead of to just return them as an array.  Simple enough: map that array with the `cell` call.

    mapStringToGameMap = mapString => mapString.split('\n').map( row => row.split('').map(chr => cell(chr)) );

Concomitantly, we need to alter the cell renderer to use the representation method, since it's no longer able to just dump in string contents.

    RenderCell = cell => '<td>' + cell.htmlRepresentation() + '</td>',

And if we look, we *now* have a semi-real game map with active cells.  For example, let's do (in a bad way) a little work to highlight the player.  First, we'll add another instance variable to the cell (**temporarily**) called isPlayer, and if it's there, we'll change the cell color.

    function cell(input) {

      var content  = input,
          isPlayer = (input === '@');

      return {
        htmlRepresentation: function() { return isPlayer? ('<span style="color:blue">' + content + '</span>') : content; }
      };

    }

This is fairly useless and terrible, but it shows that our map is actively being interpreted.

Styling inside the table cells is silly.  Let's style the table cells directly.  That means transferring the ownership of the `<td>` into the class, so let's start there.

First, the representation call changes as so (we'll change it to green so we know it's the new code working):

    htmlRepresentation: function() {
      var style = isPlayer? 'color: green' : '';
      return '<td style="' + style + '">' + content + '</td>';
    }

Next, `RenderCell` kind of doesn't need to exist anymore: all it did was wrap the `cell` class' render method in a `<td>`, and that is now provided.  So we can call the `cell` class' render method directly with an arrow function to make the access, and burn `RenderCell`.

    RenderRow = row => '<tr>' + (row.map( cell => cell.htmlRepresentation() ).join('')) + '</tr>',

This now means that the `cell` can style the table cell directly.  Let's teach it about that.

[Step 7](todo whargarbl)

----

Let's start in the CSS by creating a few rules.  One for walls, one for floors, one for players, one for doors, one for monsters (just for now,) one for items (also for now), and one for simple treasure.

Initially we'll start with walls, floors, and the player, to get the infrastructure together; then we'll do the rest.

We're going to need effectively two styles for every cell: the base and the top.  This is because the base will be the tile type (floor, lava, grass, trap, water, whatever), and there might be something (or many things) on top of it, such as a monster or treasure or whatever.

Usually, though not always, the top style will let the bottom background color through; rarely, but occasionally, it will also let the bottom foreground color through.  Each of these should be possible in our system.  It would also be nice if either could do other things, such as invoking font styling or whatever.

One way to achieve this is through CSS precedence.  Make a slightly more specific rule for top rules (we will use `<tr>` in the top rules but not in the bottom rules to get this result.)  Let the cascade handle the rest.

    #gamemap    td.wall   { background-color: dimgray;     color: silver; }
    #gamemap    td.floor  { background-color: saddlebrown; color: moccasin; }

    #gamemap tr td.player { color: cyan; }

This allows the player to stand on floor and get a brown background, but to override the floor's yellowish foreground with cyan.

Now we extend this to the other rules we want initially:

    body, html, pre       { margin: 0; padding: 0; border: 0; }

    body                  { background-color: #eee; }
    #gamemap              { border-collapse: collapse; }
    #gamemap td           { height: 1em; width: 1em; background-color: black; color: white; overflow: hidden;
                            margin: 0; padding: 0; border: 0; text-align: center; vertical-align: middle; }

    /* withhold a tr from the rule for tilekinds to force a precedence loss */
    #gamemap    td.wall     { background-color: dimgray;     color: gray; }
    #gamemap    td.floor    { background-color: saddlebrown; color: moccasin; }
    #gamemap    td.grass    { background-color: green;       color: lawngreen; }
    #gamemap    td.water    { background-color: dodgerblue;  color: darkturquoise; }
    #gamemap    td.lava     { background-color: crimson;     color: tomato; }

    /* add a tr to the rule for topkinds to force a precedence win */
    #gamemap tr td.player   { color: cyan; }
    #gamemap tr td.person   { color: white; }

    #gamemap tr td.reptile  { color: green; }
    #gamemap tr td.worm     { color: puce; }

    #gamemap tr td.item     { color: blue; }
    #gamemap tr td.treasure { color: gold; }
    #gamemap tr td.door     { color: silver; font-weight: bold; }

[Step 8](todo whargarbl)

----

# Why are items map features?

So, they aren't, is the short version.  Let's change `@` to mean "forced player starting point on this map," which suggests it's optional; let's change the monster symbols to mean "this is a forced monster spawn," treasure to mean treasure spawn, &amp;c.  Then we can just have the map scanner instantiate things it finds.  `:)`

First let's free the player from their being a map feature.

When we render a cell, currently we ask the cell what it contains.  That means the cell is responsible for knowing where things are, instead of the (currently non-existent) feature containers.  This is a mistake.

Let's make feature containers (items, monsters, traps, &amp;c.)

There is only one player, so the feature container for a player is a singleton.  Initially we will track nothing but their location, which we will default to `(0,0)` for now.  We will take an options argument to allow that default location to be overridden.

    function makePlayer(opts) {

      return {
        x: opts.x || 0,
        y: opts.y || 0
      };

    }

    var player = null;

Next let's beef up the row that creates from an '@' cell in the string map, to instead actually create a player object after validating that's the right thing to do:

    case '@' :
      if (player) { throw 'cannot have two player spawn points on one stringMap!'; }
      player     = makePlayer({ x: i, y: j });
      tileKind   = 'floor';
      tileSymbol = '.';
      break;

Next we need to add some goo in `htmlRepresentation` that checks the other sources (currently only player) before deciding what to render.

    htmlRepresentation: function() {

      var isPlayer   = (player !== null) && (i === player.x) && (j === player.y),
          uTopKind   = isPlayer? 'player' : topKind,
          uTopSymbol = isPlayer? '@'      : topSymbol,
          tclass     = (uTopKind? (uTopKind + ' ') : '') + tileKind,
          tconts     = (uTopSymbol? uTopSymbol : tileSymbol);

      return '<td class="' + tclass + '">' + tconts + '</td>';

    }

If you save and load now, the player is no longer a map feature, but there's no way to see that &hellip; until you make them move.

At this time I'm going to add a keyboard handling library called "mousetrap" to the HTML, to handle key binding.  And then we'll give this hilariously incorrect initial implementation of movement:

    Mousetrap.bind('up',    function() { player.y -= 1; Render(); });
    Mousetrap.bind('right', function() { player.x += 1; Render(); });
    Mousetrap.bind('down',  function() { player.y += 1; Render(); });
    Mousetrap.bind('left',  function() { player.x -= 1; Render(); });

We should also split the render apart from the initial string parse, and track the map and string map separately, because that matters now:

    var player  = null,
        map     = null,
        str_map = '##+#########' +  // todo: remove player, creatures, objects
                '\n#@....######' +
                '\n#...R.##...#' +
                '\n#.$...##.p.#' +
                '\n###+####...#' +
                '\n###....+..p#' +
                '\n########...#' +
                '\n############';

    function Render() {
      document.body.innerHTML = RenderMap(map);
    }

    function onStart() {
      map = mapStringToGameMap(str_map);
      Render();
    }

And now your little dude can move - though he has no bounds, like walking through walls, or off of the map.  Let's fix that next.

[Step 9](todo whargarbl)

----

# Gulp

Actually I lied.  Twice.  First I said I didn't want to set up a babel system, and then I said let's fix movement next.

The thing is, a lot of this is easier if we have full ES6, and the code is getting a little bit cumbersome; I'd like to break it up.

So let's build a quick gulp system (this implies having `node.js` set up) that uses `babel` to take modern code you write and "transpile" (convert) it to older JS that all browsers can eat; then let's also use `browserify` to package that up so that we don't have to think about it.

So.  I will assume that you have `node.js` installed, which comes with `npm`, a package manager.  If not, [go here and install it](https://nodejs.org/en/download/) before continuing.  It's quite easy.

Go into a console or command prompt, and go into your project directory.  Type `npm init`.  You can hit return and accept all those default values if you want to, or you can add descriptive text and whatever (probably just hit return until it shuts up.)

Now if you look again, it created a file called `package.json`.  That is the way that an `npm` project is defined.

Please next type `npm install`.  That should appear to do nothing.  If you look, it'll create an empty directory called `node_modules`.  That's where things that `npm` installs go.  The current behavior is because the current project's install list is empty.

Let's add a few things to that install list - specifically

1. `gulp` - an automation tool so that we can not hand-do things
1. `gulp-babel` - a gulp wrapper for a tool that converts modern JS (and other things) to portable JS
1. `browserify` - a tool that packages your various JS together
1. `del` - a tool for deleting directories

How?

`npm install --save-dev gulp gulp-babel browserify del`

(this may take a bit)

When you write `--save-dev` you're telling the system to save this as a "dev dependency."  A dependency is something `npm` will install in production.  A dev dependency is something `npm` will only install on developer machines.  So for example, if you use a documentation generation tool, it should be a dev dependency, because production doesn't need it.  (Most things should be dev dependencies.)

## Gulpfile

The `gulpfile.js` is where you keep the various steps that the build system will take for you.  We'll just start it out with saying hello, to show that it works.

Please create a `gulpfile.js` with the following contents:

    var gulp  = require('gulp');

    gulp.task('default', function() {
      console.log('hello, i am gulp');
    });

A "gulp task" is gulp's concept of a unit of work; that's something it's meant to go get done.  They always have a name, and they may have either a list of other tasks to be done first, or a function that is what they do, or both.

One task name is special: 'default', which is the task that gets run if you don't name one specifically.  Well written gulpfiles almost always have a default task.

Now go to your console and type gulp from your project directory, which should say something like

    John@SINISTAR4000 /c/projects/rogue_demo (master)
    $ gulp
    [13:51:33] Using gulpfile c:\projects\rogue_demo\gulpfile.js
    [13:51:33] Starting 'default'...
    hello, i am gulp
    [13:51:33] Finished 'default' after 5.18 ms

We now have a trivial, pointless, working gulpfile.  Let's give it a raison d'être.

First, a simplistic build system.  We'll begin with a thing that destroys a build directory that doesn't yet exist.

    var gulp  = require('gulp'),
        del   = require('del');

    gulp.task('clean', function(cb) {
      return del(['./build'], cb);
    });

    gulp.task('default', ['clean'], function() {
      console.log('hello, i am gulp');
    });

Notice that we have added an array as a second parameter to the original 'default' task.  That's the list of other tasks to get done first, which we'd mentioned earlier.

Notice also that the new gulp task takes `cb` as an argument, whereas the other did not.  This is because the gulp task needs a way to know when `del` is done before allowing it to declare itself done, and `cb` (callback) is how `del` tells `gulp` what's going on.  The task needs that to ensure that the lists of predecessor tasks are finished before the current task starts (so that we know that we've finished compiling stuff before we copy the results, or whatever.)

Now, if you go into your project directory and create a subdirectory called build, and run gulp, you'll notice it's silently destroyed, along with anything inside it.  That's our goal; we want to "nuke it from orbit, [as] that's the only way to be sure."  The 'build' directory will be automatically made and destroyed, and you shouldn't interact with it except to load its current contents in a browser.

Let's next actually have the gulp system make that directory, too.  For now it's just one directory:

    gulp.task('make-dirs', ['clean'], function(cb) {
      fs.mkdirSync('./build');
      return cb();
    });

Also, we need to add `fs` into the require list (but we don't need to `npm install` it because it's a standard part of node.)

    var gulp = require('gulp'),
        del  = require('del'),
        fs   = require('fs');

Finally, because `clean` will be run by `make-dirs`, then we can just make the `default` task run `make-dirs` and not worry about it:

    gulp.task('default', ['make-dirs'], function() {

At this point, if you make a build directory, throw stuff into it, then run `gulp`, your build directory should be emptied destroyed silently, and a new empty one made in its place.

Next let's put a lame proof-of-concept `babel` build system in place `:)`

[Step 10](todo whargarbl)

----

# Babel

This is why we're actually doing the gulp system: to get access to proper ES6, which will simplify our other code a lot.  However, one of the pieces that we want the most badly (`import`/`export`) won't be available until we do the step after this one too (`browserify`.)

Before we get started, let's give you an idea of what `babel` actually does.

You can play with `babel` in [the babel REPL](https://babeljs.io/repl/), but two quick examples are probably good enough.

## Simple `babel`

So, suppose I want to write some ES6.  I'd like to write an arrow function.  Many browsers don't do those yet.  I'd love it if I had a tool which took those and crapped out portable ES5 for me.

The code I want to write:

    [1,2,3].map(i => i * 2);

Babel will turn that into:

    "use strict";

    [1, 2, 3].map(function (i) {
      return i * 2;
    });

And that will run pretty much anywhere.

## Less simple `babel`

The code I want to write:

    class TwoFourSix {

      asArray() { [1,2,3].map(i => i * 2); }

    };

Babel will turn that into:

    "use strict";

    var _createClass = (function () { function defineProperties(target, props) { for (var i = 0; i < props.length; i++) { var descriptor = props[i]; descriptor.enumerable = descriptor.enumerable || false; descriptor.configurable = true; if ("value" in descriptor) descriptor.writable = true; Object.defineProperty(target, descriptor.key, descriptor); } } return function (Constructor, protoProps, staticProps) { if (protoProps) defineProperties(Constructor.prototype, protoProps); if (staticProps) defineProperties(Constructor, staticProps); return Constructor; }; })();

    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

    var TwoFourSix = (function () {
      function TwoFourSix() {
        _classCallCheck(this, TwoFourSix);
      }

      _createClass(TwoFourSix, [{
        key: "asArray",
        value: function asArray() {
          [1, 2, 3].map(function (i) {
            return i * 2;
          });
        }
      }]);

      return TwoFourSix;
    })();

## Okay so we get `babel`; let's use it

Let's get started.

Like the other steps, we'll start with the smallest thing we can call a success, then build on that incrementally more confidently.

First, a config file for `babel`.  We'll keep config files in a new directory called `config`, so this file is `config/babel.json`.

    {
      "presets": ["babel-preset-es2015"]
    }

Next, we'll load that in the gulpfile:

    var babel_cfg = require('./config/babel.json');

Now we can make a `gulp task` that handles this transformation for us.

    gulp.task('babel', function() {
      return gulp.src('./src/*.js')
        .pipe( babel(babel_cfg) )
        .pipe( gulp.dest('./build/js') );
    });

We should also update the `make-dirs` task to create a subdir `js` of the build directory for us:

    ['./build', './build/js'].map(dir => fs.mkdirSync(dir));

So, `gulp.src` fetches files from disk according to a `glob` pattern; `.pipe` sends whatever we have to a next step, `babel()` is the thing that causes the actual transformation, and `gulp.dest` writes things to disk (note it's going to a subdirectory of the build directory.)

This task loads files from our `./src` directory, which is notable because that directory doesn't exist yet.  Let's make it, and put a simple JS in with the first one-liner example `[1,2,3].map(i => i * 2);` in it, under the name `bootstrap.js`.

And now if you run gulp, you'll see that the build directory contains a subdirectory called `js`, which contains a file called `bootstrap`, that has the ES5 version of our more modern code in it.  Hurrah!  We can write better stuff.

Next, it's `browserify` time.

----

## What was that `browserify` bit?

So, the loading of scripts situation in Javascript is just a mess.  There are several competing historical third party standards; no browser supports those directly; there is an ES6 standard; no browser supports that directly outside experimental mode; and `babel` can translate the ES6 system back to the `node.js`-relevant third party ES5 system.

So if we load a packager for that specific ES5 system ("commonjs,") then `babel` can handle the rest, and we're good to go.

There are a lot of such packagers, such as `webpack`, `jspm`, `requirejs`, `qoopido.demand`, and so on.  We're going to go with `browserify`, because compared to the others it's simpler.

It is still, admittedly, a little complex.

What these things actually do is accept a list of places they should start looking, then dig through them looking for `require()` calls, then load those targets, then dig, then ... , building out a physical dependency tree; then they package up all those dependencies inside some boilerplate that all makes it work from inside a single file.  And pow: there's your one script with its nine libraries and fifty subscripts in a single file.

The reason we care about this is that our current single file is getting ungainly, stupid, and overcomplicated, and it barely even exists yet.  It's chopping time.

First, we need a minimalist file to package to show a running package.  We create a `src/app.js` that just contains `window.alert('app!');` so that we can see the bundle running once it starts.

Next, a `gulp task`, so that we don't actually have to think about it at all.

    gulp.task('browserify', ['babel'], function() {

      return browserify(browserify_cfg, { "debug" : !production })

        .require("./build/js/app.js", { "expose" : "app" })

        .bundle()
        .on("error", errorHandler)
        .pipe(source("bundle.js"))
        .pipe(gulp.dest('./build/js'));

    });

What this does:

1. `browserify()` is the packaging system we discussed
1. The `production` bit mostly just tells it whether to write sourcemaps (not today thanks)
1. The `.require` tells it to use `app.js` as a module, and the `expose` tells it to make that module available to others under that name
1. `.bundle()` is the "ok it's time to go" flag
1. `source("bundle.js")` is using a tool called "vinyl source streams" to provide the single packaged up file (weird name, I know)
1. And the `.dest` puts it on disk under that name in the build location.

Notice also that there's a `browserify_cfg` there.  That's a JSON config file, like the `babel` one, and we ought to set it up too.  So go make a `config/browserify.json`, and toss this inside:

    {
      "entries" : ["./build/js/bootstrap.js"]
    }

> "What is bootstrap.js?"

It's a new file.  Its only purpose is to invoke `app` so that browserify will build a `require()` tree there.  This is an artifact of browsers not having modern ES6 packaging support.  Just make a `src/bootstrap.js` and toss in `var App = require('app');`, and we're good.

Update your gulpfile's default task to point at `browserify`, and continue:

    gulp.task('default', ['browserify']);

Now, in theory, we have a running packaging system.  We'll need to add a line to our HTML to point at the bundle now:

    <script defer type="text/javascript" src="build/js/bundle.js"></script>

And if you reload, you should get an alert box with your existing roguelike.  Which means your `browserify` and `babel` setup are done. `:D`

> "What the hell was that for?"

That's coming next `:D`

