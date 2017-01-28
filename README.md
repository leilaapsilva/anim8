anim8
=====

Biblioteca de animação para [LÖVE](http://love2d.org).

[![Build Status](https://travis-ci.org/kikito/anim8.png?branch=master)](https://travis-ci.org/kikito/anim8)
[![Coverage Status](https://coveralls.io/repos/github/kikito/anim8/badge.svg?branch=master)](https://coveralls.io/github/kikito/anim8?branch=master)

Para fazer animações mais facilmente, anim8 divide o processo em duas etapas: primeiro você cria um *grid", que é capaz 
de criar *frames* (quadros) de forma fácil e rápida. Então, você usa o grid para criar uma ou mais *animações*.


Compatibilidade com LÖVE 
==================


Uma vez que anim8 usa funções gráficas de LÖVE, e nós mudamos de versão a versão, você deve escolher a versão 
de anim8 que seja compatível com seu LÖVE.

* A versão atual de anim8 é v2.1. É compatível com LÖVE 0.9.x e 0.10.x
* A última versão de anim8 compatível com LÖVE 0.8.x era [anim8 v2.0](https://github.com/kikito/anim8/tree/v2.0.0).

Exemplo
=======

```
local anim8 = require 'anim8'

local image, animation

function love.load()
  image = love.graphics.newImage('path/to/image.png')
  local g = anim8.newGrid(32, 32, image:getWidth(), image:getHeight())
  animation = anim8.newAnimation(g('1-8',1), 0.1)
end

function love.update(dt)
  animation:update(dt)
end

function love.draw()
  animation:draw(image, 100, 200)
end
```

Este demo transforma esse spritesheet:

Você pode ver um exemplo mais elaborado em [demo branch](https://github.com/kikito/anim8/tree/demo).

![1945](http://kikito.github.io/anim8/1945.png)

Em vários objetos animados:

![1945](http://kikito.github.io/anim8/anim8-demo.gif)

Explicação 
===========

Grids
-----

Grids possuem apenas dois propósitos: Construir grupos de quadros do mesmo tamanho o mais facilmente possível. Para fazer isso, eles precisam saber apenas duas coisas: o tamanho de cada quadro e o tamanho da imagem em que eles serão aplicados. Cada tamanho é uma largura e altura, e estes são os 4 primeiros parâmetros de @anim8.newGrid@.

Grids são apenas uma forma conveniente de obter frames de um sprite. Assume-se que os frames são distribuídos em linhas e colunas. Frame 1,1 é a primeira linha, primeira coluna. 

Aqui está como se cria um grid:

`anim8.newGrid(frameWidth, frameHeight, imageWidth, imageHeight, left, top, border)`:

* `frameWidth` and `frameHeight` are the dimensions of the animation *frames* - each of the individual "sub-images" that compose the animation. They are usually the same size as your character (so if the character is
    32x32 pixels, `frameWidth` is 32 and so is `frameHeight`)
* `imageWidth` and `imageHeight` are the dimensions of the image where all the frames are. In LÖVE, they can be obtained by doing `image:getWidth()` and `image:getHeight()`.
* `left` and `top` are optional, and both default to 0. They are "the left and top coordinates of the point in the image where you want to put the origin of coordinates of the grid". If all the frames in your grid are
  the same size, and the first one's top-left corner is 0,0, you probably won't need to use `left` or `top`.
* `border` is also an optional value, and it also defaults to zero. What `border` does is allowing you to define "gaps" between your frames in the image. For example, imagine that you have frames of 32x32, but they
  have a 1-px border around each frame. So the first frame is not at 0,0, but at 1,1 (because of the border), the second one is at 1,33 (for the extra border) etc. You can take this into account and "skip" these borders.

To see this a bit more graphically, here are what those values mean for the grid which contains the "submarine" frames in the demo:

![explanation](http://kikito.github.io/anim8/anim8-explanation.png)

Grids only have one important method: `Grid:getFrames(...)`.

`Grid:getFrames` accepts an arbitrary number of parameters. They can be either numbers or strings.

* Each two numbers are interpreted as quad coordinates. This way, `grid:getFrames(3,4)` will return the frame in column 3, row 4 of the grid. There can be more than just two: `grid:getFrames(1,1, 1,2, 1,3)` will return the frames in {1,1}, {1,2} and {1,3} respectively.
* Using numbers for long rows is tedious - so grids also accept strings. The previous row of 3 elements, for example, can be also expressed like this: `grid:getFrames(1,'1-3')` . Again, there can be more than one string (`grid:getFrames(1,'1-3', '2-4',3)`) and it's also possible to combine them with numbers (`grid:getFrames(1,4, 1,'1-3')`)

But you will probably never use getFrames directly. You can use a grid as if it was a function, and getFrames will be called. In other words, given a grid called `g`, this:

    g:getFrames('2-8',1, 1,2)

Is equivalent to this:

    g('2-8',1, 1,2)

This is very convenient to use in animations.

Let's consider the submarine in the previous example. It has 7 frames, arranged horizontally.

If you make its grid start on its first frame (using `left` and `top`), you can get its frames like this:

                           -- frame, image,    offsets, border
    local gs = anim8.newGrid(32,98, 1024,768,  366,102,   1)

    local frames = gs('1-7',1)

However that way you will get a submarine which "emerges", then "suddenly disappears", and emerges again. To make it look more natural, you must add some animation frames "backwards", to give the illusion
of "submersion". Here's the complete list:

    local frames = gs('1-7',1, '6-2',1)



Animations
----------

Animations are groups of frames that are interchanged every now and then.

`local animation = anim8.newAnimation(frames, durations, onLoop)`:

* `frames` is an array of frames (Quads in LÖVE argot). You could provide your own quad array if you wanted to, but using a grid to get them is very convenient.
* `durations` is a number or a table. When it's a number, it represents the duration of all frames in the animation. When it's a table, it can represent different durations for different frames. You can specify durations for all frames individually, like this: `{0.1, 0.5, 0.1}` or you can specify durations for ranges of frames: `{['3-5']=0.2}`.
* `onLoop` is an optional parameter which can be a function or a string representing one of the animation methods. It does nothing by default. If specified, it will be called every time an animation "loops". It will have two parameters: the animation instance, and how many loops have been elapsed. The most usual value (apart from none) is the
string 'pauseAtEnd'. It will make the animation loop once and then pause and stop on the last frame.

Animations have the following methods:

`animation:update(dt)`

Use this inside `love.update(dt)` so that your animation changes frames according to the time that has passed.

`animation:draw(image, x,y, angle, sx, sy, ox, oy, kx, ky)`

Draws the current frame in the specified coordinates with the right angle, scale, offset & shearing. These parameters work exactly the same way as in [love.graphics.draw](https://love2d.org/wiki/love.graphics.draw).
The only difference is that they are properly recalculated when the animation is flipped horizontally, vertically or both. See `getFrameInfo` below for more details.

`animation:gotoFrame(frame)`

Moves the animation to a given frame (frames start counting in 1).

`animation:pause()`

Stops the animation from updating (@animation:update(dt)@ will have no effect)

`animation:resume()`

Unpauses an animation

`animation:clone()`

Creates a new animation identical to the current one. The only difference is that its internal counter is reset to 0 (it's on the first frame).

`animation:flipH()`

Flips an animation horizontally (left goes to right and viceversa). This means that the frames are simply drawn differently, nothing more.

Note that this method does not create a new animation. If you want to create a new one, use the `clone` method.

This method returns the animation, so you can do things like `local a = anim8.newAnimation(g(1,'1-10'), 0.1):flipV()`

`animation:flipV()`

Flips an animation vertically. The same rules that apply to `flipH` also apply here.

`animation:pauseAtEnd()`

Moves the animation to its last frame and then pauses it.

`animation:pauseAtStart()`

Moves the animation to its first frame and then pauses it.

`animation:getDimensions()`

Returns the width and height of the current frame of the animation. This method assumes the frames passed to the animation are all quads (like the ones
created by a grid).

`animation:getFrameInfo(x,y, r, sx, sy, ox, oy, kx, ky)`

This functions returns the parameters that would be passed to `love.graphics.draw` when drawing this animation:
`frame, x, y, r, sx, sy, ox, oy, kx, ky`.

* `frame` is the currently active frame for the animation (usually a quad produced by a grid)
* `x,y` are the same coordinates passed as parameter to `getFrame` (there are no changes)
* `r` is the same angle passed to `getFrame`, with no changes unless it is `nil`, in which case it becomes 0.
* `sx,sy` are the scale values, with their sign changed if the animation is flipped vertically or horizontally
* `ox,oy` are the offset values, with the width or height properly substracted if the animation is flipped. 0 is used as a initial value for these calculations if nil was passed.
* `kx,ky` are the shearing factors, changed depending on the flip status.

The `getFrame` method can be used when working with [spriteBatches](https://love2d.org/wiki/SpriteBatch). Here's how it can be used for adding & setting the corresponding quad in a spritebatch:

``` lua
local id = spriteBatch:add(animation:getFrameInfo(x,y,r,sx,sy,ox,oy,kx,ky))

...

spriteBatch:set(id, animation:getFrameInfo(x,y,r,sx,sy,ox,oy,kx,ky))
```

You can see an example of this in the [spritebatch-demo branch](https://github.com/kikito/anim8/tree/spritebatch-demo).


Installation
============

Just copy the anim8.lua file wherever you want it. Then require it wherever you need it:

    local anim8 = require 'anim8'

Please make sure that you read the license, too (for your convenience it's included at the beginning of the anim8.lua file).

Specs
=====

This project uses [busted](http://olivinelabs.com/busted/) for its specs. If you want to run the specs, you will have to install it first. Then just execute the following from the root folder:

    busted
