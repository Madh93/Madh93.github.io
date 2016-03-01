---
layout: project
title:  "Algoritmo de búsqueda A*"
date:   2016-03-01 00:26:23
release: 15 de Diciembre de 2014
language: Qt/C++
author: Miguel Hernández, José Alberto Mena García, Adexe Sabina Pérez
task: Programación
source: https://github.com/Madh93/Qt5-AStarPathfinding
categories:
- proyectos
- C++
- Qt
- Algoritmos
img: algoritmo-a-estrella.jpg
carousel:
- algoritmo-a-estrella01.jpg
- algoritmo-a-estrella02.jpg
---

#### ¿Qué es?

Una implementación del [algoritmo de búsqueda A\*](https://es.wikipedia.org/wiki/Algoritmo_de_b%C3%BAsqueda_A*) desarrollado en Qt.

Suponiendo que tenemos un sistema cuyo fin es encontrar la trayectoria, o camino mínimo dentro de unos rangos asignados, evitando cualquier tipo de obstáculos. La ambientación del sistema muestra la necesidad de un taxi en encontrar a un peatón a lo largo de la ciudad, teniendo que evitar edificios, parques y zonas en obras.

#### Características

El programa se divide en dos partes. Primero se debe generar un escenario, de tamaño MxN. Posteriormente se colocan los obstáculos, añadiéndolos de forma manual, o de forma aleatoria.

<video autoplay="" controls="" loop="" class="video-js vjs-default-skin col-lg-12" data-setup="{}">
  <source src="https://zippy.gfycat.com/TornSingleChevrotain.webm" type="video/webm">
</video>

Cabe destacar que se puede añadir obstáculos con clic primario, y quitar con clic secundario sobre el elemento.

Por último, se establece el punto de inicio (el taxi) y el final (el peatón), y se genera la solución óptima.

<video autoplay="" controls="" loop="" class="video-js vjs-default-skin col-lg-12" data-setup="{}">
  <source src="https://fat.gfycat.com/CarelessJointBluegill.webm" type="video/webm">
</video>
<br>

#### Uso

Desde Github se puede descargar el código fuente. Para poder compilarlo y posteriormente ejecutarlo es necesario tener [Qt 5 o superior](http://www.qt.io/download/). Ha sido testeado en las versiones 5.4 y 5.5.

#### Opinión personal

Este fue mi primer proyecto utilizando Qt, y no solo eso, el primero en utilizar una interfaz gráfica, y pienso que el resultado final sigue sin decepcionarme a día de hoy. Fue un proyecto que hice junto a otros dos amigos, y salió adelante gracias a la buena compenetración, y es que cada uno aportó en lo que más destacaba.