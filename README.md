anim8
=====

Biblioteca de animação para [LÖVE](http://love2d.org).

[**Versão original em inglês**].(https://github.com/kikito/anim8)

[![Build Status](https://travis-ci.org/kikito/anim8.png?branch=master)](https://travis-ci.org/kikito/anim8)
[![Coverage Status](https://coveralls.io/repos/github/kikito/anim8/badge.svg?branch=master)](https://coveralls.io/github/kikito/anim8?branch=master)

Para fazer animações mais facilmente, anim8 divide o processo em duas etapas: primeiro você cria um *grid*, que é capaz 
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

* `frameWidht`and `frameHeight`são as dimensões dos *frames* da animação - cada uma das "sub-imagens" que compõem a animação. São usualmente do mesmo tamanho do seu personagem (então se o personagem é de 32x32 pixels, `frameWidth` e `frameHeight` são 32) 
* `imageWidth` e `imageHeight`são as dimensões da imagem onde todos os frames estão. Em LÖVE, elas podem ser obtidas com `image:getWidht()` e `image:getHeight()`.
* `left`e `top`são opcionais, e o valor padrão de ambos é 0. São "as coordenadas superior-esquerdas do ponto na imagem onde você quer colocar a origem das coordenadas do grid". Se todos os frames no seu grid são do mesmo tamanho, e o primeiro no canto superior-esquerdo é 0,0, você provavelmente não precisa usar `left` ou `top`.
* `border` é também um valor opcional, e também possui valor padrão zero. O que `border`faz é permitir que você defina "lacunas" entre seus frames imagem. Por exemplo, imagine que você tem frames de 32x32, mas eles tem uma borda de 1 pixel ao redor de cada frame. Então o primeiro frame não está em 0,0, mas em 1,1 (por causa da borda), o segundo está em 1,33 (por causa da borda extra) etc. Você pode levar isso em conta e "pular" essas bordas.

Para ver isso um pouco mais gráficamente, aqui está o que significam esses valores para o grid que contém os frames do "submarino" na demo:

![explanation](http://kikito.github.io/anim8/anim8-explanation.png)

Grids possuem apenas um método importante: `Grid:getFrames(...)`.

`Grid:getFrames` aceita um número arbitrário de parâmetros. Eles podem ser números ou strings.

* Cada dois números são interpretados como coordenadas de um quadro. Deste modo, `grid:getFrames(3,4)`irá returnar o frame na coluna 3, linha 4 do grid. Podem ser mais do que apenas dois: `grid:getFrames(1,1, 1,2, 1,3)` retornará os frames em {1,1}, {1,2} e {1,3} respectivamente.
* Usar números para linhas longas é tedioso - assim, os grids também aceitam strings. A linha anterior de 3 elementos, por exemplo, também pode ser expressa assim: `grid:getFrames(1, '1-3')`. Novamente, pode ser mais do que uma string (`grid:getFrames(1,'1-3', '2-4',3)`) e isso também é possível para combiná-los com números (`grid:getFrames(1,4, 1, '1-3')`)

Mas você provavelmente nunca usará getFrames diretamente. Você pode usar um grid como se fosse uma função, e getFrames será chamado. Em otras palavras, dado um grid chamado `g`, isto:

    g:getFrames('2-8',1, 1,2)
    
É equivalente a isto:   

    g('2-8',1, 1,2)

Isso é muito conveniente para uso em animações.

Vamos considerar o submarino do exemplo anterior. Ele possui 7 frames, arranjados horizontalmente. 

Se você fizer seu grid começar em seu primeiro frame (usando `left`e `top`), você pode obter seus frames assim:

                            -- frame, image,    offsets, border
    local gs = anim8.newGrid(32,98, 1024,768,  366,102,   1)

    local frames = gs('1-7',1)
    
No entanto, dessa forma você terá um submarino que "emerge", então "de repente desaparece", e emerge novamente. Para torná-lo mais natural, você deve adicionar alguns frames de animação "para trás", para dar a ilusão de "submersão". Aqui está a lista completa:

    local frames = gs('1-7',1, '6-2',1)

Animações
----------

Animações são grupos de frames que são trocados de vez em quando.

`local animation = anim8.newAnimation(frames, durations, onLoop)`:

* `frames` é um vetor de frames (Quads em gíria de LÖVE). Você poderia forneceer o seu próprio array de quadros se você quisesse, mas usar um grid para obtê-los é muito conveniente.
* `durantions` é um número ou uma tabela. Quando e um número, representa a duração de todos os frames na animação. Quando é uma tabela, pode representar durações diferentes para frames diferentes. Você pode especificar durações para todos os frames individualmente, assim: `{0.1, 0.5, 0.1}`ou você pode especificar durações para intervalos de frames: `{['3-5']=0.2}`.
* `onLoop`é um parâmetro opcional que pode ser uma função ou uma string representando um dos métodos da animação. Não faz nada por padrão. Se especificado, será chamado toda vez que uma animação fizer o loop. Terá dois parâmetros: a instância da animação, e quantos loops foram decorridos. O valor mais usual (além de nenhum) é a string 'pauseAtEnd'. Fará a animação fazer o loop uma vez e então pausar e parar no último frame.

Animações possuem os seguintes métodos:

`animation:update(dt)`

Use isso dentro de `love.update(dt)` para que sua animação mude de frames de acordo com o tempo que passou.

`animation:draw(image, x,y, angle, sx, sy, ox, oy, kx, ky)`

Desenha o frame atual nas coordenadas especificadas com o ângulo certo, escala, deslocamento e corte. Estes parâmetros funcionam exatamente da mesma forma que em [love.graphics.draw](https://love2d.org/wiki/love.graphics.draw).
A núnica diferente é que eles são recalculados corretamente quando a animação é invertida horizintalmente, verticalmente ou ambos. Veja `getFrameInfo` abaixo para mais detalhes.

`animation:gotoFrame(frame)`

Move a animação para um determinado frame (frames começam a ser contados em 1).

`animation:pause()`

Para a animação da atualização (@animation:update(dt)@ não terá efeito)

`animation:resume()`

Retoma uma animação

`animation:clone()`

Cria uma nova animação idêntica à atual. A única diferença é que seu contator interno é resetado para 0 (está no primeiro frame).

`animation:flipH()`

Inverte uma animação horizontalmente (esquerda vai para direita e vice-versa). Isso significa que os frames são simplesmente desenhados de forma diferente, nada mais.

Note que este método não cria uma nova animação. Se você quer criar uma nova, use o método `clone`.

Esse método retorna a animação, então você pode fazer coisas como `local a = anim8.newAnimation(g(1,'1-10'), 0.1):flipV()`

`animation:flipV()`

Inverte a animação verticalmente As mesmas regras aplicadas a `flipH`também se aplicam aqui.

`animation:pauseAtEnd()`

Move a animação para seu último frame e então a pausa.

`animation:pauseAtStart()`

Move a animação para o seu primeiro frame e então a pausa.

`animation:getDimensions()`

Retorna a larura e altura do frame atual da animação. Este método assumo que os frames passados para a animação são todos quadros (como o criado por um grid).

`animation:getFrameInfo(x,y, r, sx, sy, ox, oy, kx, ky)`

Esta função retorna os parâmetros que seriam passados para `love.graphics.draw` ao desenhar esta animação:
`frame, x, y, r, sx, sy, ox, oy, kx, ky`.

* `frame`é o frame atualmente ativo para a animação (geralmente um quadro produzido por um grid)
* `x,y`são as mesmas coordenadas passadas como parâmetro para `getFrame` (não há mudanças)
* `r`é o mesmo ângulo passado para `getFrame`, sem mudanças a menos que seja `nil`, caso em que se torna 0
* `sx, sy`são os valores de escala, com seu sinal alterado se a animação é virada verticalmente ou horizontalmente
* `ox,oy`são os valores de deslocamento, com a largura ou altura corretamente subtraída se a animação é invertida. 0 é usado como um valor inicial para estes cálculos se nil foi passado.
* `kx,ky`são os fatores de corte, alterados dependendo do estado de flip (iversão).

O método `getFrame`pode ser usado quando trabalhando com [spriteBatches](https://love2d.org/wiki/SpriteBatch). Veja como ele pode ser usado para adicionar e configurar o quadro correspondente em um spritebatch:

``` lua
local id = spriteBatch:add(animation:getFrameInfo(x,y,r,sx,sy,ox,oy,kx,ky))

...

spriteBatch:set(id, animation:getFrameInfo(x,y,r,sx,sy,ox,oy,kx,ky))
```

Você pode ver um exemplo disso em [spritebatch-demo branch](https://github.com/kikito/anim8/tree/spritebatch-demo).

Instalação
============

Apenas copie o arquivo anim8.lua para onde você quiser. Em seguida, adicione um require onde você precisar:

    local anim8 = require 'anim8'
    
Certifique-se de que leu a licença, também (para sua conveniência ela está incluída no início do arquivo anim8.lua).

Especificações 
=====

Este projeto usa [busted](http://olivinelabs.com/busted/) para suas especificações. Se você quiser executar as especificações, você terá que instalá-lo primeiro. Em seguida, basta executar o seguinte a partir da pasta raíz:

    busted
