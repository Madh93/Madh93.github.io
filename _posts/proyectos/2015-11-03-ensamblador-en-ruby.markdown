---
layout: project
title:  "ENSAMBLADOR EN RUBY"
date:   2015-11-03 00:47:12
release: 10 de Mayo de 2015
language: Ruby
author: Miguel Hernández
task: Programación
source: https://github.com/Madh93/rips
categories:
- proyectos
- Ruby
img: ensamblador-en-ruby.jpg
carousel:
- ensamblador-en-ruby01.jpg
- ensamblador-en-ruby02.jpg
---

####¿Qué es?

Un ensamblador escrito en Ruby para la [CPU monociclo de 16-bit realizada en verilog](http://www.migueldhdez.me/realizacion-de-una-cpu-simple/).

Como había comentado en el proyecto de la CPU simple, para comprobar el funcionamiento de la CPU era necesario introducir las instrucciones "código máquina", en unos y ceros básicamente. Nadie programa en binario, es una locura, para ello existen los ensambladores. 

Un ensamblador no es más que un programa intermedio entre el software y el hardware, que traduce el código fuente escrito en lenguaje ensamblador a código máquina.

####Características

El ensamblador, se llama Rips debido a algunas semejanzas con [MIPS](https://en.wikipedia.org/wiki/MIPS_instruction_set). Este cuenta con las características más básicas de un ensamblador adaptadas a nuestra CPU:

- 19 instrucciones
- 16 registros ($0-$15)
- 4 puetos de E/S (@0-@3)
- Uso de etiquetas y comentarios

<video autoplay="" controls="" loop="" class="video-js vjs-default-skin col-lg-12" data-setup="{}">
  <source src="http://zippy.gfycat.com/BareDirectAmericancreamdraft.webm" type="video/webm">
</video>

El funcionamiento es muy sencillo. La idea es leer código en ensamblador (llamémoslo instrucciones .rips) y generar código binario. El ensamblador se encarga de analizar la morfología y sintaxis de las instrucciones .rips.

####Uso

Desde Github se puede descargar el código fuente y algunos ejemplos. También se puede instalar como gema desde la terminal o [RubyGems](https://rubygems.org/gems/rips). Rips ha sido testeado en las versiones 1.9.3, 2.0, 2.1 y 2.2 de Ruby y no requiere de ninguna dependencia extra a parte del propio intérprete de Ruby.

Para consultar en detalle el funcionamiento está disponible la [documentación oficial](http://www.rubydoc.info/gems/rips/0.1.1).

####Opinión personal

Cuando me dieron la posibilidad de poder realizar un ensamblador para mi propia CPU, aunque suene a tópico, no lo dude ni un instante y me puse a ello desde el primer momento. Era algo que me llamaba mucho la atención.

Durante la realización de la CPU mi compañero y yo, nos dábamos cuenta que depurar código en binario era una locura. Realizar un ensamblador, era algo que nos facilitaría mucho la vida, y así fue.

La decisión de utilizar Ruby fue personal. Ruby es uno de mis lenguajes favoritos, consigo resultados rápidos y me divierto mucho programando en él.