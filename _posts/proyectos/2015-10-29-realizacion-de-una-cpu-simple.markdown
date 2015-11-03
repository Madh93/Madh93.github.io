---
layout: project
title:  "REALIZACIÓN DE UNA CPU SIMPLE"
date:   2015-10-29 23:54:45
release: 10 de Mayo de 2015
language: Verilog
author: Miguel Hernández, José Alberto Mena García
task: Programación
source: https://github.com/Madh93/scpu/tree/cc24d6b685c251c87f29c7cc732201d59178be3d
categories:
- proyectos
- Verilog
img: realizacion-de-una-cpu-simple.jpg
carousel:
- realizacion-de-una-cpu-simple01.jpg
- realizacion-de-una-cpu-simple02.jpg
---

####¿Qué es?

Se trata de una CPU monociclo de 16-bit realizada en Verilog, un ejemplo perfecto para entender desde lo más básico cual es el verdadero funcionamiento de un microprocesador.

Tradicionalmente una CPU ejecuta una instrucción tras otra. Estas instrucciones son almacenadas en una memoria de instrucciones. La unidad de control de la CPU es la responsable de extraer cada instrucción, decodificarla, y ejecutarla de manera secuencial mediante un contador de programa. Además de la memoria de instrucciones y un contador de programa, se suele disponer de un conjunto de registros donde almacenar los datos y una ALU (unidad aritmético-lógica) encargada de las operaciones más elementales.

####Características

La CPU cuenta con una memoria de instrucciones con capacidad de 1024 instrucciones, un banco de 16 registros de 8 bits, una ALU con operaciones como asignación, suma, resta, and, or, diferencia, etc. y cuatro puertos de entrada/salida.

<a data-rel="prettyPhoto" href="/assets/img/projects/content/realizacion-de-una-cpu-simple01.jpg" data-animate="fadeInUp">
  ![Diseño de la CPU](/assets/img/projects/content/realizacion-de-una-cpu-simple01.jpg)
</a>
Se han especificado una quincena de instrucciones: operaciones de ALU, operaciones de carga de datos, operaciones de entrada/salida, saltos incondicionales, saltos condicionale y saltos a subrutina con retorno. Todas ellas utilizan un Opcode variable entre 4 y 6 bits.

Para comprobar el correcto funcionamiento de la CPU, escribimos un pequeño código en binario de una calculadora que utilizaba algunas de las instrucciones descritas anteriormente, como las operaciones aritméticas, los saltos y la interacción entre la entrada y salida.

Sin embargo, como codificar instrucciones en binario es una lata, decidimos implementar un [ensamblador para nuestra CPU](http://www.migueldhdez.me/ensamblador-en-ruby/).

####Uso

Desde Github se puede descargar el código fuente. El proyecto ha sido probado con el software oficial Altera Quartus II v12.1 bajo Linux y la [fpga Altera DE1](http://www.terasic.com.tw/cgi-bin/page/archive.pl?No=83).

<a data-rel="prettyPhoto" href="/assets/img/projects/content/realizacion-de-una-cpu-simple02.jpg" data-animate="fadeInUp">
  ![FPGA Altera DE1](/assets/img/projects/content/realizacion-de-una-cpu-simple02.jpg)
</a>

####Opinión personal

A día de hoy, este proyecto lo sigo considerando uno de los mayores retos. Ya no solo por el hecho de realizar mi propia CPU, que se dice pronto, sino por tener que trabajar de una manera totalmente diferente a la que estaba acostumbrado.

Verilog es un lenguaje de descripción de hardware, y realmente no tiene nada que ver con cualquier lenguaje de programación convencional, la forma de enfocar el problema, y de como este se puede resolver es distinta.

Por supuesto, la recompensa de haberlo finalizado supera todo el esfuerzo realizado.