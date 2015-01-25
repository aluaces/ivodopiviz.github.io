---
layout: post
title: SDL 2.0 Migration Guide
---

##Introducción

Luego de muchos años de desarrollo, ¡SDL 2.0 ha sido finalmente liberada!

Estamos bastante orgullosos de esto, y nos gustaría que los juegos que utilizan SDL 1.2 migraran rápido. Como sabemos que puede ser una tarea intimidante, este documento es una simple guía de cómo migrar a la nueva librería. Creemos que descubrirás que no es tan difícil como piensas y, muchas veces, simplemente tendrás que reemplazar algunos llamados a función o deshacer "hacks" en tu código causados por deficiencias de la versión 1.2.

Creemos que SDL 2.0 va a hacerte feliz, por sus características nuevas y por una mejor experiencia con respecto a SDL 1.2. Este documento no trata de cubrir todas las funcionalidades nuevas de SDL2 - que son muchas - sino sólo aquellas cosas que necesitas para empezar a trabajar. Una vez que hayas portado tu código, no dudes en chequear todo lo nuevo,probablemente quieras utilizarlo en tus aplicaciones.

##Vistazo de características nuevas

Estas son las características nuevas más importantes de SDL 2.0

*Aceleración 3D por hardware
*Soporte de OpenGL 2.0+ con múltiples perfiles (core, compatibilidad, depuración, robusto, etc)
*Soporte de OpenGL ES
*Soporte de ventanas múltiples
*Soporte de pantallas múltiples
*Soporte de dispositivos de audio múltiples
*Soporte de Android e iOS
*API 2D simple que puede utilizar Direct3D, OpenGL, OpenGL ES o renderizado por software.
*Soporte Force Feedback en Windows, Mac OSX X y Linux
*Soporte XInput y XAudio2 en Windows
*Operaciones atómicas
*Manejo de energía (expone batería restante, etc)
*Shaped windows // not sure how to translate this
*Audio 32 bit (int y float)
*API de controladores de juego simplificada (¡la API de Joystick sigue existiendo también!)
*Soporte de control táctil (multitoques, gestos, etc)
*Mejorado el soporte de pantalla completa
*Mejorado el soporte de teclado (scancodes vs keycodes, etc)
*Ventanas de mensaje
*Soporte de portapapeles
*Soporte básico Drag'n'Drop
*Soporte para entrada IME y unicode

*Poderoso sistema de macros para asserts
*Cambiada la licencia LGPL por zlib
*Muchas de las molestias de 1.2 ya no existen
*¡Y mucho más!

La página de Introducción posee una lista más exhaustiva de las características ofrecidas por SDL en general (incluyendo las ya provistas por 1.2)