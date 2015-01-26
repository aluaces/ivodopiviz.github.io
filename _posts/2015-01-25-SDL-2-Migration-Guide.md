---
layout: post
title: Guía de migración a SDL 2.0
---

##Introducción

Luego de muchos años de desarrollo, ¡SDL 2.0 ha sido finalmente liberada!

Estamos bastante orgullosos de esto, y nos gustaría que los juegos que utilizan SDL 1.2 migraran rápido. Como sabemos que puede ser una tarea intimidante, este documento es una simple guía de cómo migrar a la nueva librería. Creemos que descubrirás que no es tan difícil como piensas y, muchas veces, simplemente tendrás que reemplazar algunos llamados a función o deshacer "hacks" en tu código causados por deficiencias de la versión 1.2.

Creemos que SDL 2.0 va a satisfacerte tanto por sus características nuevas y por una mejor experiencia con respecto a SDL 1.2. Este documento no trata de cubrir todas las funcionalidades nuevas de SDL2 - que son muchas - sino sólo aquellas cosas que necesitas para empezar a trabajar. Una vez que hayas portado tu código, no dudes en chequear todo lo nuevo: probablemente quieras utilizarlo en tus aplicaciones.

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

Así que probablemente si algo así:

	SDL_WM_SetCaption("My Game Window", "game");
	SDL_Surface *screen = SDL_SetVideoMode(640, 480, 0, SDL_FULLSCREEN | SDL_OPENGL);

Ahora debería ser:

	SDL_Window *screen = SDL_CreateWindow("My Game Window",
							SDL_WINDOWPOS_UNDEFINED,
							SDL_WINDOWPOS_UNDEFINED,
							640, 480,
							SDL_WINDOW_FULLSCREEN | SDL_WINDOW_OPENGL);

Notarás que es bastante similar a 1.2. La diferencia es que ahora es posible tener varias ventanas (si quieres) y tienes más control sobre ellas. SDL_WM_SetCaption ya no existe, porque es necesatio que cada ventana pueda tener un título propio (puedes cambiarlo usando SDL_SetWindowTitle()) y que sea posible darle una posición específica. En este caso utilizamos SDL_WINDOWPOS_UNDEFINED porque no nos preocupa dónde se ubique la ventana. También es posible utilizar SDL_WINDOWPOS_CENTERED.

Además, es posible especificar una pantalla: SDL2 te permite trabajar con equipos con monitores múltiples. No es necesario que te preocupes por eso por ahora, sin embargo.

Ahora que tu ventana esta lista, hablemos de estrategia. SDL2 sigue teniendo SDL_Surface pero, de ser posible, deberías utilizar SDL_Texture. Ahora las "surface" siempre se encuentran en RAM y son controladas por el CPU, así que es mejor evitarlas. SDL2 tiene una API de renderizado nueva, pensada para juegos 2D simples pero por sobre todo para utilizar el GPU y la memoria de video lo más posible. Incluso si sólo deseas mostrar en pantalla tu render por software, tienes el siguiente beneficio: de ser posible se utilizará OpenGL o Direct3D lo cual significa que tendrás "blits" más rápidos, reescalado de pantalla "gratis" y será posible utilizar el Steam Overlay.

El procedimiento ahora es así:

Como ya dijimos, en vez de SDL_SetVideoMode() utilizamos SDL_CreateWindow(). ¿Pero qué resolución utilizamos? Si tu juego está fijo a 640x480, por ejemplo, seguro pasaba que ciertos monitores no podían poner ese tamaño en pantalla completa y en ventana tu juego se veía demasiado pequeño. Con SDL2 es posible solucionar eso.

Ya no utilizamos SDL_ListModes(). Hay algo parecido en SDL2 (llamar SDL_GetDisplayMode() en un bucle tantas veces como diga SDL_GetNumDisplayModes()), pero en vez de eso vamos a utilizar una nueva característica llamada "escritorio a pantalla completa" que básicamente le dice a SDL: "dame la pantalla completa y no cambies la resolución". Para nuestro juego a 640x480, sería algo así:

	SDL_Window *sdlWindow = SDL_CreateWindow(title,
								SDL_WINDOWPOS_UNDEFINED,
								SDL_WINDOWPOS_UNDEFINED,
								0, 0,
								SDL_WINDOW_FULLSCREEN_DESKTOP);

Si te fijas, no especificamos 640 o 480... escritorio a pantalla completa te da justamente la pantalla completa e ignora las dimensiones que le pases. La ventana de tu juego debería aparecer instantáneamente en vez de esperar a que el monitor cambie de resolución y con esto vamos a utilizar el GPU para escalar todo a la resolución del escritorio. Esto es generalmente lo mejor y más rápido en caso de que una pantalla LCD esté simulando correr a una resolución menor. Como "bonus": no es necesario redimensionar el resto de las ventanas que estén corriendo.

Ahora necesitamos un contexto de renderizado.

	SDL_Renderer *renderer = SDL_CreateRenderer(sdlWindow, -1, 0);

El "renderer" abstrae todos los detalles de cómo se dibuja en la ventana. Puede estar utilizando Direct3D, OpenGL, OpenGL ES o "software surfaces", dependiendo del sistema, pero tu código siempre será el mismo. Es posible, sin embargo, forzar un renderer específico. Si deseas forzar la sincronización con vblank para reducir artefactos gráficos, puedes utilizar SDL_RENDERER_PRESENTVSYNC en vez de cero como tercer parámetro. No deberías especificar SDL_WINDOW_OPENGL directamente: si SDL_CreateRenderer() decide utilizar OpengGL, va a actualizar la ventana de manera apropiada.

Ahora que sabes cómo funciona esto, es posible hacerlo en un sólo paso utilizando SDL_CreateWindowAndRenderer(). Un ejemplo simple:

	SDL_Window *sdlWindow;
	SDL_Renderer *sdlRenderer;
	SDL_CreateWindowAndRenderer(0, 0, SDL_WINDOW_FULLSCREEN_DESKTOP, &sdlWindow, &sdlRenderer);

Asumiendo que ninguna de las funciones falló (¡siempre hay que chequear NULLs!), estamos listos para comenzar a dinujar en la pantalla. Empecemos por "limpiarla" usando color negro:

	SDL_SetRenderDrawColor(sdlRenderer, 0, 0, 0, 255);
	SDL_RenderClear(sdlRenderer);
	SDL_RenderPresent(sdlRenderer);

Probablemente imagines cómo funciona esto: dibujamos con color negro (r, g y b en cero, alpha opaco), limpiamos la ventana completa y presentamos la ventana limpia en la pantalla. De la misma manera que antes usaba SDL_UpdateRect() o SDL_Flip(), ahora se utiliza SDL_RenderPresent().

Una cosa más: Como estamos utilizando SDL_WINDOW_FULLSCREEN_DESKTOP, no sabemos realmente el tamaño de la pantalla en la que estamos dibujando. Afortunadamente, no necesitamos saberlo. Una de las mejores cosas de 1.2 es que era posible decirle "Quiero una ventana de 640x480 y no me importa cómo lo logres", incluso si eso implicaba mostrar una ventana pequeña centrada en una resolución más grande.

En 2.0, la API de render nos permite hacer lo siguiente:

	SDL_SetHint(SDL_HINT_RENDER_SCALE_QUALITY, "linear");  // hacemos que el render escalado se vea mejor
	SDL_RenderSetLogicalSize(sdlRenderer, 640, 480);

Esto nos permite tener tener una resolución lógica independiente de la resolución final de presentación. Gracias a esto, en vez de tratar de hacer que el sistema funcione a la resolución que nosotros renderizamos, cambiamos la resolución de renderizado para que funcione acorde al sistema. En mi pantalla de 1920x1200, esta aplicación cree que está trabajando con una pantalla 640x480, pero SDL está utilizando el GPU para escalar el renderizado y utilizar todos los pixels. Es importante notar que 640x480 y 1920x1200 no tienen la misma relación de aspecto: SDL tratará de escalar lo más que pueda e incluirá un marco de color para la diferencia.

Ya podemos empezar a dibujar de verdad.

###Si tu juego sólo necesita dibujar frames completos en la pantalla

Un caso particular para juegos de la vieja escuelta que utilizan renderizado por software: la aplicación quiere dibujar cada pixel por sí misma y luego plasmar todo eso directamente en la pantalla en un solo "blit". Ejemplos de este tipo de juegos serían Doom, Duke Nukem 3D, entre otros.

Para esto, sólo necesitas una SDL_Texture que va a representar la pantalla. Hagamos eso para nuestro juego a 640x480:

	sdlTexture = SDL_CreateTexture(sdlRenderer,
								SDL_PIXELFORMAT_ARGB8888,
								SDL_TEXTUREACCESS_STREAMING,
								640, 480);

Esto representa una textura en la GPU. La idea es generar cada frame guardando pixels en esta textura, dibujar la textura en la ventana y presentar la misma en la pantalla. SDL_TEXTUREACCESS_STREAMING le indica a SDL que el contenido de esta textura cambiará de manera frecuente.

Anteriormente, probablemente hayas utilizado una SDL_Surface dedicada a la pantalla y luego llamaras SDL_Flip() para mostrarla. Ahora es posible crear un SDL_Surface en RAM en vez de utilizar la que te hubiera dado SDL_SetVideoMode() o incluso simplemente utilizar malloc() para generar un bloque de pixels al cual puedas escribir. Idealmente, tus buffers deberían contener pixels RGBA, pero de todas maneras es posible realizar conversiones.

	extern Uint32 *myPixels;  // esto puede ser surface->pixels, un buffer de malloc() o lo que sea.

Al final del frame, necesitamos actualizar nuestra textura de la siguiente manera:

	SDL_UpdateTexture(sdlTexture, NULL, myPixels, 640 * sizeof (Uint32));

Esto va a "subir" nuestros pixels a memoria de GPU. En vez de NULL, es posible especificar un área en caso de utilizar "dirty rectangles", pero lo más probable es que no sea necesario gracias al hardware moderno. El último parametro es el "pitch" -el número de bytes desde el principio de una fila hasta la siguiente- y si tenemos en cuenta que en este ejemplo utilizamos un búfer RGBA lineal, el "pitch" es 640 veces 4 (r, g, b, a).

Ahora pongamos esta textura en pantalla:

	SDL_RenderClear(sdlRenderer);
	SDL_RenderCopy(sdlRenderer, sdlTexture, NULL, NULL);
	SDL_RenderPresent(sdlRenderer);

Y eso es todo. SDL_RenderClear() limpia el contenido del framebuffer (por las dudas de que, digamos, el Steam Overlay lo haya modificado en el cuadro anterior), SDL_RenderCopy() mueve el contenido de la textura al framebuffer (y gracias a SDL_RenderSetLogicalSize(), lo hará de manera escalada/centrada como si el monitor fuera de 640x480) y SDL_RenderPresent() lo presentará en la pantalla.

###If your game wants to blit surfaces to the screen