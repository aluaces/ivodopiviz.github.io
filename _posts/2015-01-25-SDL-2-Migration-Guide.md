---
layout: post
title: Guía de migración a SDL 2.0
---

##Introducción

Luego de muchos años de desarrollo, ¡SDL 2.0 ha sido finalmente liberada!

Estamos bastante orgullosos de esto, y nos gustaría que los juegos que utilizan SDL 1.2 migraran rápido. Como sabemos que puede ser una tarea intimidante, este documento es una simple guía de cómo migrar a la nueva librería. Creemos que descubrirás que no es tan difícil como piensas y, muchas veces, simplemente tendrás que reemplazar algunos llamados a función o deshacer "hacks" en tu código causados por deficiencias de la versión 1.2.

Creemos que SDL 2.0 va a hacerte feliz, por sus características nuevas y por una mejor experiencia con respecto a SDL 1.2. Este documento no trata de cubrir todas las funcionalidades nuevas de SDL2 - que son muchas - sino sólo aquellas cosas que necesitas para empezar a trabajar. Una vez que hayas portado tu código, no dudes en chequear todo lo nuevo,probablemente quieras utilizarlo en tus aplicaciones.

##Vistazo de características nuevas

Estas son las características nuevas más importantes de SDL 2.0

* Aceleración 3D por hardware
* Soporte de OpenGL 2.0+ con múltiples perfiles (core, compatibilidad, depuración, robusto, etc)
* Soporte de OpenGL ES
* Soporte de ventanas múltiples
* Soporte de pantallas múltiples
* Soporte de dispositivos de audio múltiples
* Soporte de Android e iOS
* API 2D simple que puede utilizar Direct3D, OpenGL, OpenGL ES o renderizado por software.
* Soporte Force Feedback en Windows, Mac OSX X y Linux
* Soporte XInput y XAudio2 en Windows
* Operaciones atómicas
* Manejo de energía (expone batería restante, etc)
* Shaped windows // not sure how to translate this
* Audio 32 bit (int y float)
* API de controladores de juego simplificada (¡la API de Joystick sigue existiendo también!)
* Soporte de control táctil (multitoques, gestos, etc)
* Mejorado el soporte de pantalla completa
* Mejorado el soporte de teclado (scancodes vs keycodes, etc)
* Ventanas de mensaje
* Soporte de portapapeles
* Soporte básico Drag'n'Drop
* Soporte para entrada IME y unicode

* Poderoso sistema de macros para asserts
* Cambiada la licencia LGPL por zlib
* Muchas de las molestias de 1.2 ya no existen
* ¡Y mucho más!

La página de Introducción posee una lista más exhaustiva de las características ofrecidas por SDL en general (incluyendo las ya provistas por 1.2)

##Más información

Los mejores lugares para ver más información son:

* esta wiki :)
* los tests incluídos con SDL, en el directorio test/ (ver online)
* las lista de correo de SDL

##Migrando de SDL 1.2 a 2.0

###Consideraciones generales

No hay una capa de compatibilidad en SDL2. Si una API cambió en 2.0, quiere decir que cambiamos o removimos el código antiguo donde era necesario. Si te limitas a compilar tu aplicación basada en 1.2 contra las cabeceras de 2.0, probablemente no compile. Este documento intentará guiarte con respecto a los cambios más importantes y algunos problemas con los que probablemente tropieces.

No existe SDL_main! Bueno, en realidad existe, y ahora es lo que siempre se supuso que debía ser: un prqueño trozo de código que oculta la differencia entre main() y WinMain() en Windows. No posee código de inicialización y es completamente opcional- Esto significa que ahora es posible utilizar SDL sin sobreescribir tu main, lo cual es bueno para plugins que utilicen SDL, o lenguajes script que utilicen SDL como un módulo. Todo lo que en 1.2 se hacía en SDL_main ahora se hace en SDL_Init(), donde corresponde.

El "paracaídas" de SDL no existe más. Lo que en 1.2 se llamaba SDL_INIT_NOPARACHUTE es la opción por defecto ahora. Causaba problemas en caso de que hubiera problemas en threas que no fueran el principal así como con aplicaciones que utilizara su propio sistema de señales/manejo de excepciones. Lamentablemente, algunas plataformas no limpian el sistema de video a pantalla completa en caso de errores críticos, así que deberías configurar un manejador de crashes propio o llamar a SDL_Quit() en un atexit() de ser necesario.
Ten en cuenta que en plataformas Unix, SDL sigue interceptando SIGINT y lo convierte en un evento SDL_QUIT.

##Video

###Configurando la nueva API de video

La API de video es el cambio más importante con respecto a 1.2. Los requerimientos de un sistema de video han cambiado muchos desde los '90s. Para poder hacer uso del hardware moderno y nuevas funcionalidades de los SO, tuvimos que cambiar la API de video casi de manera completa.

No te preocupes, la API nueva es muy buena y una vez comprendas las diferencias vas a estar muy satisfecho con las cosas nuevas que trae consigo. Pero dejemos eso para más tarde.

Las buenas noticias: si tu juego utilizaba OpenGL, probablemente no tengas mucho que toca: con cambiar las llamadas a funcion a sus equivalentes de SDL2 alcanza.

Para gráficos 2D, SDL 1.2 ofrecía algo llamado "surfaces", que eran búfers de pixels en memoria. Incluso la misma pantalla era una "surface" y SDL ofrecía funciones para copiar ("blit") pixels entre "surfaces", convirtiendo entre formatos de ser necesario. La mayor parte del tiempo te encontrabas trabajando con el CPU y la memoria de sistema en vez de utilizar el GPU y la memoria de video. Esto cambia con SDL 2.0; ahora se utiliza aceleración por hardware la mayor parte del tiempo y es por esto que hemos tenido que cambiar la API.

Si tu juego es 2D, lo más probable es que hayas seguido alguna de las siguientes tre rutas con respecto al renderizado. Vamos a analizarlas a todas, pero empecemos por el principio.

¿Recuerdas a SDL_SetVideoMode()? No existe más. SDL 2.0 te permite tener varias ventanas, así que esa función ya no tenía mucho sentido.

Así que probablemente tenías algo así: