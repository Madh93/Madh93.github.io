---
layout: project
title:  "Implementación de FTP"
date:   2015-10-04 01:33:41
release: 16 de Mayo de 2014
language: C/C++
author: Miguel Hernández
task: Programación
source: https://github.com/Madh93/ftp
categories:
- proyectos
- C++
- C
img: implementacion-de-ftp.jpg
carousel:
- implementacion-de-ftp01.jpg
- implementacion-de-ftp02.jpg
- implementacion-de-ftp03.jpg
---

####¿Qué es?

Se trata de una implementación del protocolo [FTP](https://es.wikipedia.org/wiki/File_Transfer_Protocol) escrito en C++ pero utilizando las librerías estándar de C, que sirve de ejemplo de una aplicación cliente-servidor utilizando el concepto de sockets. 

Los sockets no son más que un mecanismo que permite la comunicación de datos entre dos dispositivo conociendo su dirección del protocolo y el número de puerto correspondiente.

El protocolo FTP es un protocolo de red que trabaja sobre la capa de aplicación, el cual permite la transferencia de ficheros entre un servidor y un cliente.

####Características

Para tener un protocolo FTP, lo básico es disponer de la posibilidad de descargar archivos desde el servidor, como subirlos a él. Estas acciones se pueden realizar tanto mediante modo "activo" como "pasivo".

<video autoplay="" controls="" loop="" class="video-js vjs-default-skin col-lg-12" data-setup="{}">
  <source src="http://zippy.gfycat.com/FocusedDistantHuman.webm" type="video/webm">
</video>

Además de esto, el usuario es posible navegar por el servidor ftp, moviéndose por los directorios del mismo, creando nuevas carpetas, o incluso eliminando algunos archivos.

<video autoplay="" controls="" loop="" class="video-js vjs-default-skin col-lg-12" data-setup="{}">
  <source src="http://zippy.gfycat.com/WeeklyMadFattaileddunnart.webm" type="video/webm">
</video>
<br>

####Uso

Desde Github se puede descargar el código fuente. En el mismo repositorio están las instrucciones para compilar los archivos. Cabe mencionar, que para comprobar el funcionamiento lo correcto sería lanzar los clientes desde diferentes directorios al servidor.

####Opinión personal

Este proyecto, fue el proyecto final de una asignatura de Redes. El objetivo era entender el concepto de los sockets, los cuales son indispensables en las comunicaciones. 

No fue un tema fácil de entender, sin embargo la realización de un protocolo FTP como ejemplo fue indispensable para entender la el funcionamiento de los sockets, y ya de paso profundizar en el funcionamiento de FTP (existiendo los conocidos modo activo y modo pasivo).