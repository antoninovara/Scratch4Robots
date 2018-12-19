This is the old Kurt 1.4 documentation. The compiler stuff has yet to be
rewritten for Kurt 2.0, so we ignore it for now.

The rest is outdated. Seriously, ignore this.



## Using the Compiler

*Skip to [Using the Library](#using-the-library) if you only want to use Kurt to make your own awesome Python stuff*

    Scratch project  <--->  folder structure of scripts and images

If you install using easy_install, pip, or setup.py, the script `kurtc.py` should automatically be in your path. Try running it from the command line:

    $ kurtc.py

You should see a screenful of help text. If not, try installing again using options 1 or 2. Alternatively, the script can be found under `util/kurtc.py`.

There are two commands: `compile`, and `decompile`.


### Decompile

    scratch project -> folder structure with project contents

Exports all the scripts, images, variables, lists in a Scratch project to separate files, with a folder for each sprite. Puts everything in a folder named `<project name> files`. (Think a Scratch [project summary](#note1) on steroids.)

Just pass the path to your project file:

    $ kurtc.py decompile tests/game.sb

And get a folder structure a bit like this:

    game files/
        00 Stage/                                   [each sprite has its own directory]
            backgrounds/
                01 background1.png
            backgrounds.txt
            lists/
            scripts/
            variables.txt
        01 ScratchCat/
            costumes/
                01 costume1.png                     [export to PNG or JPG format files,
                02 costume2.jpg                      import from most formats]
            costumes.txt                            [costume details, rotation centers]
            lists/                                  [.txt file for each list]
            scripts/                                [.txt files: block plugin syntax]
                01 when green flag clicked.txt
            variables.txt                           [variable = value, one per line]
        notes.txt
        thumbnail.png

Most of the subfolders are optional when compiling.

Notes:

* Scripts are in [Block Plugin](wiki.scratch.mit.edu/wiki/Block_Plugin) ("scratchblocks") syntax.
* `costumes.txt` is entirely optional — you should probably ignore it.
* Lists have one item per line.
* Variable files have one variable per line, like `variable = value`.
* Any un-numbered files/folders will be added last.


### Compile

    folder structure -> scratch project

Takes a folder structure generated by decompile as an argument, and compiles all the images and scripts back into a .sb file.

Try decompiling the provided `game.sb`, edit one of the scripts, and then compiling it!

Again, just pass it the path to your project folder:

    $ kurtc.py compile "tests/game files"


### Notes & Restrictions

* Sprite information such as position isn't saved — so make sure you set it in a "when green flag clicked" script (which is probably good practice anyway).

* Parser SyntaxErrors currently report line numbers -2 the actual file. This is a bug.

* Take care with the "length of" block: strings aren't dropdowns, lists are

        length of [Hello!]      // string
        length of [list v]      // list

* Variable names (and possibly other values, such as broadcasts) **can't**:

    * contain special identifiers (like `end`, `if`, etc.)
    * have trailing whitespace
    * contain special characters, rather obviously: like any of `[]()<>` or equals `=`
    * be named after a block, eg. a variable called "wait until" ([a screenshot](http://cl.ly/image/3z0X3O1O0m1w))




## Using the Library

**Getting Started** | [Scripts](#scripts) | [Images](#images)

Here's a quick getting started — grab a Python interpreter (Python's `>>>` prompt — just type `python` into your terminal, or load up IDLE), and have a go!

You'll probably just want to use the provided `ScratchProjectFile` and `ScratchSpriteFile` classes. Load them by passing the path to the file to their constructor and use their provided `.save()` methods.

You can import just these classes them using:

    >>> from kurt.files import *

Load a file (you'll find a preview file, `game.sb`, saved in the `tests` directory; but feel free to try it with any Scratch project file).

Just pass in the absolute or relative path to the file:

    >>> project = ScratchProjectFile("tests/game.sb")

You can reload the file at any time with `.load()`.

Inspect project:

    >>> project.info['author']
    u'blob8108'
    >>> project.stage
    <Stage(Stage)>

List fields on object:

    >>> project.stage.fields.keys()
    ['volume', 'hPan', 'sprites', 'lists', 'name', 'obsoleteSavedState',
    'color', 'media', 'variables', 'bounds', 'submorphs', 'zoom', 'isClone',
    'flags', 'costume', 'scripts', 'owner', 'tempoBPM', 'vPan', 'properties', 'sceneStates']

Access fields using dot notation:

    >>> project.stage.tempoBPM
    60
    >>> project.stage.sprites
    [<Sprite(ScratchCat)>]

List sprites:

    >>> project.sprites             # alias for `project.stage.sprites`
    [<Sprite(ScratchCat)>]
    >>> cat = project.sprites[0]
    >>> cat.name
    'ScratchCat'

Can also index sprites list by name:

    >>> cat = project.sprites['ScratchCat']

Most of the objects you're interested in, like `Stage` and `Sprite`, inherit from `UserObject`. You can use `.fields.keys()` to see the
available fields on one of these objects.

`FixedObjects` like `Rectangle` have a `.value` property to access their value.

    >>> project.stage.bounds
    Rectangle([0, 0, 480, 360])
    >>> project.stage.bounds.value
    [0, 0, 480, 360]

Inline objects, such as `int` and `bool`, are converted transparently to their Pythonic counterparts. `Array` and `Dictionary` are converted to `list` and `dict`, too.

Make changes:

    >>> cat.vars # {u'vx': 0.0}
    >>> cat.vars['vx'] = 100

Save:

    >>> project.save()

Now re-open the project with Scratch!

Everything should, of course, work perfectly; if you do have any problems, please send me an email or [file an issue on Github](https://github.com/blob8108/kurt/issues/new), and I'll take a look! (:


### Scripts
A list of scripts can be found on the `scripts` property of both sprites and the stage.

    >>> cat.scripts
    [Script(Point(23, 36.0), [
        Block('EventHatMorph', 'Scratch-StartClicked'),
        Block('xpos:', 0),
        Block('doForever',  [
                Block('doIf',
                    Block('keyPressed:', 'right arrow'),
                    [
                        Block('changeVariable', u'vx', <#changeVar:by:>, 2),
                    ]),
                Block('doIf',
                    Block('keyPressed:', 'left arrow'),
                    [
                        Block('changeVariable', u'vx', <#changeVar:by:>, -2),
                    ]),
                Block('changeVariable', u'vx', <#setVar:to:>,
                    Block('*',
                        Block('readVariable', u'vx'),
                    0.80000000000000004),
                ),
                Block('changeXposBy:',
                    Block('readVariable', u'vx'),
                ),
            ])])]

Use the `to_block_plugin` method to print them nicely:

    >>> print cat.scripts[0].to_block_plugin()
    when green flag clicked
    set x to (0)
    forever
        if <key [right arrow v] pressed?>
            change [vx v] by (2)
        end
        if <key [left arrow v] pressed?>
            change [vx v] by (-2)
        end
        set [vx v] to ((vx) * (0.8))
        change x by (vx)
    end

This is identical to `scratchblocks` format, so you can paste them straight into the Scratch forums or wiki. [See here](http://dl.dropbox.com/u/9598124/blocksplugin.html#when%2520green%2520flag%2520clicked%250Aset%2520x%2520to%2520%280%29%250Aforever%250A%2520%2520%2520%2520if%2520%253Ckey%2520%255Bright%2520arrow%2520v%255D%2520pressed%3F%253E%250A%2520%2520%2520%2520%2520%2520%2520%2520change%2520%255Bvx%2520v%255D%2520by%2520%282%29%250A%2520%2520%2520%2520end%250A%2520%2520%2520%2520if%2520%253Ckey%2520%255Bleft%2520arrow%2520v%255D%2520pressed%3F%253E%250A%2520%2520%2520%2520%2520%2520%2520%2520change%2520%255Bvx%2520v%255D%2520by%2520%28-2%29%250A%2520%2520%2520%2520end%250A%2520%2520%2520%2520set%2520%255Bvx%2520v%255D%2520to%2520%28%28vx%29%2520%2A%2520%280.8%29%29%250A%2520%2520%2520%2520change%2520x%2520by%2520%28vx%29%250Aend).

See [Scripts](https://github.com/blob8108/kurt/wiki/Script) on the kurt wiki.


### Images
You can find costumes under a sprite's `costumes` property (similarly for stage `backgrounds`).

    >>> cat.costumes # [<ImageMedia(costume1)>, <ImageMedia(costume2)>]
    >>> image = cat.costumes[0]

Save to an external file:

    >>> image.save("scratch_cat.png")


