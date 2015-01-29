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

* [Poderoso sistema de macros para asserts](https://wiki.libsdl.org/CategoryAssertions)
* Cambiada la licencia LGPL por zlib
* Muchas de las molestias de 1.2 ya no existen
* ¡Y mucho más!

La página de Introducción posee una lista más exhaustiva de las características ofrecidas por SDL en general (incluyendo las ya provistas por 1.2)

##Más información

Los mejores lugares para ver más información son:

* esta wiki :)
* los tests incluídos con SDL, en el directorio test/ ([ver online](http://hg.libsdl.org/SDL/file/default/test))
* la [lista de correo](http://lists.libsdl.org/listinfo.cgi/sdl-libsdl.org) de SDL

##Migrando de SDL 1.2 a 2.0

###Consideraciones generales

No hay una capa de compatibilidad en SDL2. Si una API cambió en 2.0, quiere decir que cambiamos o removimos el código antiguo donde era necesario. Si te limitas a compilar tu aplicación basada en 1.2 contra las cabeceras de 2.0, probablemente no compile. Este documento intentará guiarte con respecto a los cambios más importantes y algunos problemas con los que probablemente tropieces.

No existe SDL_main! Bueno, en realidad existe, y ahora es lo que siempre se supuso que debía ser: un prqueño trozo de código que oculta la differencia entre main() y WinMain() en Windows. No posee código de inicialización y es completamente opcional- Esto significa que ahora es posible utilizar SDL sin sobreescribir tu main, lo cual es bueno para plugins que utilicen SDL, o lenguajes script que utilicen SDL como un módulo. Todo lo que en 1.2 se hacía en SDL_main ahora se hace en SDL_Init(), donde corresponde.

El "paracaídas" de SDL no existe más. Lo que en 1.2 se llamaba SDL_INIT_NOPARACHUTE es la opción por defecto ahora. Causaba problemas en caso de que hubiera problemas en threas que no fueran el principal así como con aplicaciones que utilizara su propio sistema de señales/manejo de excepciones. Lamentablemente, algunas plataformas no limpian el sistema de video a pantalla completa en caso de errores críticos, así que deberías configurar un manejador de crashes propio o llamar a [SDL_Quit()](https://wiki.libsdl.org/SDL_Quit) en un atexit() de ser necesario.
Ten en cuenta que en plataformas Unix, SDL sigue interceptando SIGINT y lo convierte en un evento [SDL_QUIT](https://wiki.libsdl.org/SDL_EventType).

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

Notarás que es bastante similar a 1.2. La diferencia es que ahora es posible tener varias ventanas (si quieres) y tienes más control sobre ellas. SDL_WM_SetCaption ya no existe, porque es necesatio que cada ventana pueda tener un título propio (puedes cambiarlo usando [SDL_SetWindowTitle()](https://wiki.libsdl.org/SDL_SetWindowTitle)) y que sea posible darle una posición específica. En este caso utilizamos SDL_WINDOWPOS_UNDEFINED porque no nos preocupa dónde se ubique la ventana. También es posible utilizar SDL_WINDOWPOS_CENTERED.

Además, es posible especificar una pantalla: SDL2 te permite trabajar con equipos con monitores múltiples. No es necesario que te preocupes por eso por ahora, sin embargo.

Ahora que tu ventana esta lista, hablemos de estrategia. SDL2 sigue teniendo [SDL_Surface](https://wiki.libsdl.org/SDL_Surface) pero, de ser posible, deberías utilizar [SDL_Texture](https://wiki.libsdl.org/SDL_Texture). Ahora las "surface" siempre se encuentran en RAM y son controladas por el CPU, así que es mejor evitarlas. SDL2 tiene una API de renderizado nueva, pensada para juegos 2D simples pero por sobre todo para utilizar el GPU y la memoria de video lo más posible. Incluso si sólo deseas mostrar en pantalla tu render por software, tienes el siguiente beneficio: de ser posible se utilizará OpenGL o Direct3D lo cual significa que tendrás "blits" más rápidos, reescalado de pantalla "gratis" y será posible utilizar el Steam Overlay.

El procedimiento ahora es así:

Como ya dijimos, en vez de SDL_SetVideoMode() utilizamos [SDL_CreateWindow()](https://wiki.libsdl.org/SDL_CreateWindow). ¿Pero qué resolución utilizamos? Si tu juego está fijo a 640x480, por ejemplo, seguro pasaba que ciertos monitores no podían poner ese tamaño en pantalla completa y en ventana tu juego se veía demasiado pequeño. Con SDL2 es posible solucionar eso.

Ya no utilizamos SDL_ListModes(). Hay algo parecido en SDL2 (llamar [SDL_GetDisplayMode()](https://wiki.libsdl.org/SDL_GetDisplayMode) en un bucle tantas veces como diga [SDL_GetNumDisplayModes()](https://wiki.libsdl.org/SDL_GetNumDisplayModes)), pero en vez de eso vamos a utilizar una nueva característica llamada "escritorio a pantalla completa" que básicamente le dice a SDL: "dame la pantalla completa y no cambies la resolución". Para nuestro juego a 640x480, sería algo así:

	SDL_Window *sdlWindow = SDL_CreateWindow(title,
								SDL_WINDOWPOS_UNDEFINED,
								SDL_WINDOWPOS_UNDEFINED,
								0, 0,
								SDL_WINDOW_FULLSCREEN_DESKTOP);

Si te fijas, no especificamos 640 o 480... escritorio a pantalla completa te da justamente la pantalla completa e ignora las dimensiones que le pases. La ventana de tu juego debería aparecer instantáneamente en vez de esperar a que el monitor cambie de resolución y con esto vamos a utilizar el GPU para escalar todo a la resolución del escritorio. Esto es generalmente lo mejor y más rápido en caso de que una pantalla LCD esté simulando correr a una resolución menor. Como "bonus": no es necesario redimensionar el resto de las ventanas que estén corriendo.

Ahora necesitamos un contexto de renderizado.

	SDL_Renderer *renderer = SDL_CreateRenderer(sdlWindow, -1, 0);

El "renderer" abstrae todos los detalles de cómo se dibuja en la ventana. Puede estar utilizando Direct3D, OpenGL, OpenGL ES o "software surfaces", dependiendo del sistema, pero tu código siempre será el mismo. Es posible, sin embargo, forzar un renderer específico. Si deseas forzar la sincronización con vblank para reducir artefactos gráficos, puedes utilizar SDL_RENDERER_PRESENTVSYNC en vez de cero como tercer parámetro. No deberías especificar SDL_WINDOW_OPENGL directamente: si [SDL_CreateRenderer()](https://wiki.libsdl.org/SDL_CreateRenderer) decide utilizar OpengGL, va a actualizar la ventana de manera apropiada.

Ahora que sabes cómo funciona esto, es posible hacerlo en un sólo paso utilizando [SDL_CreateWindowAndRenderer()](https://wiki.libsdl.org/SDL_CreateWindowAndRenderer). Un ejemplo simple:

	SDL_Window *sdlWindow;
	SDL_Renderer *sdlRenderer;
	SDL_CreateWindowAndRenderer(0, 0, SDL_WINDOW_FULLSCREEN_DESKTOP, &sdlWindow, &sdlRenderer);

Asumiendo que ninguna de las funciones falló (¡siempre hay que chequear NULLs!), estamos listos para comenzar a dinujar en la pantalla. Empecemos por "limpiarla" usando color negro:

	SDL_SetRenderDrawColor(sdlRenderer, 0, 0, 0, 255);
	SDL_RenderClear(sdlRenderer);
	SDL_RenderPresent(sdlRenderer);

Probablemente imagines cómo funciona esto: dibujamos con color negro (r, g y b en cero, alpha opaco), limpiamos la ventana completa y presentamos la ventana limpia en la pantalla. De la misma manera que antes usaba SDL_UpdateRect() o SDL_Flip(), ahora se utiliza [SDL_RenderPresent()](https://wiki.libsdl.org/SDL_RenderPresent).

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

Anteriormente, probablemente hayas utilizado una [SDL_Surface](https://wiki.libsdl.org/SDL_Surface) dedicada a la pantalla y luego llamaras SDL_Flip() para mostrarla. Ahora es posible crear una [SDL_Surface](https://wiki.libsdl.org/SDL_Surface) en RAM en vez de utilizar la que te hubiera dado SDL_SetVideoMode() o incluso simplemente utilizar malloc() para generar un bloque de pixels al cual puedas escribir. Idealmente, tus buffers deberían contener pixels RGBA, pero de todas maneras es posible realizar conversiones.

	extern Uint32 *myPixels;  // esto puede ser surface->pixels, un buffer de malloc() o lo que sea.

Al final del frame, necesitamos actualizar nuestra textura de la siguiente manera:

	SDL_UpdateTexture(sdlTexture, NULL, myPixels, 640 * sizeof (Uint32));

Esto va a "subir" nuestros pixels a memoria de GPU. En vez de NULL, es posible especificar un área en caso de utilizar "dirty rectangles", pero lo más probable es que no sea necesario gracias al hardware moderno. El último parametro es el "pitch" -el número de bytes desde el principio de una fila hasta la siguiente- y si tenemos en cuenta que en este ejemplo utilizamos un búfer RGBA lineal, el "pitch" es 640 veces 4 (r, g, b, a).

Ahora pongamos esta textura en pantalla:

	SDL_RenderClear(sdlRenderer);
	SDL_RenderCopy(sdlRenderer, sdlTexture, NULL, NULL);
	SDL_RenderPresent(sdlRenderer);

Y eso es todo. [SDL_RenderClear()](https://wiki.libsdl.org/SDL_RenderClear) limpia el contenido del framebuffer (por las dudas de que, digamos, el Steam Overlay lo haya modificado en el cuadro anterior), [SDL_RenderCopy()](https://wiki.libsdl.org/SDL_RenderCopy) mueve el contenido de la textura al framebuffer (y gracias a [SDL_RenderSetLogicalSize()](https://wiki.libsdl.org/SDL_RenderSetLogicalSize, lo hará de manera escalada/centrada como si el monitor fuera de 640x480) y [SDL_RenderPresent()](https://wiki.libsdl.org/SDL_RenderPresent) lo presentará en la pantalla.

###Si tu juego requiere "blitear" surfaces a la pantalla

En este ejemplo suponemos que tu juego basado en SDL 1.2 carga imágenes desde archivo y las guarda en [SDLSurfaces](https://wiki.libsdl.org/SDL_Surface), probablemente tratando de ubicarlas en memoria de video utilizando SDL_HWSURFACE. Las cargas una vez y las "bliteas" una y otra vez en el framebuffer tanto como sea necesario, en general sin modificarlas. Como haríamos en un plataformero 2D simple, por poner un ejemplo. En líneas generales, si utilizas tus surfaces como "sprites", probablemente este sea tu caso de uso.

Puedes crear texturas individuales (surfaces ubicadas en memoria de GPU) de la misma manera que hicimos para la textura grande:

	sdlTexture = SDL_CreateTexture(sdlRenderer,
								SDL_PIXELFORMAT_ARGB8888,
								SDL_TEXTUREACCESS_STATIC,
								myWidth, myHeight);

En este caso, utilizamos **SDL_TEXTUREACCESS_STATIC**, porque vamos a "subir" los pixels sólo una vez y no repetidas veces. Pero una una mejor solución sería:

	sdlTexture = SDL_CreateTextureFromSurface(sdlRenderer, mySurface);

De esta manera, cargas tu [SDL_Surface](https://wiki.libsdl.org/SDL_Surface) de la manera usual pero al final creamos una textura a partur de la misma. Una vez que tienes tu [SDL_Texture](https://wiki.libsdl.org/SDL_Texture) es posible librerar la surface original.

Para este momento, tu juego utilizando SDL 1.2 tendría unas cuantas [SDL_Surfaces](https://wiki.libsdl.org/SDL_Surface) que dibujarias en la surface de pantalla utilizando [SDL_BlitSurface()](https://wiki.libsdl.org/SDL_BlitSurface) para componer el framebuffer final, eventualmente utilizando SDL_Flip() para presentarlo en pantalla. En SDL 2.0 tendrás [SDL_Textures](https://wiki.libsdl.org/SDL_Texture) que vas a copiar a tu Renderer con [SDL_RenderCopy()](https://wiki.libsdl.org/SDL_RenderCopy) para componer el mismo framebuffer, presentándolo en pantalla a su vez con [SDL_RenderPresent()](https://wiki.libsdl.org/SDL_RenderPresent). Así de simple. Si nuncas modificas dichas texturas, probablemente notes un aumento dramático en tu framerate.

###Si necesitas ambas

La cosa se complica un poco si necesitas blitear surfaces y modificar pixels individuales en el framebuffer. Leer y escribir datos desde texturas de GPU puede ser terriblemente costoso en cuanto a rendimiento; en líneas generales lo mejor es enviar los datos siempre en el mismo sentido. En este caso, la mejor idea es mantener todo en modo software hasta que esté listo para ser enviado a la pantalla. Para esto, vamos a combinar las dos técnicas anteriores.

La buena noticia es que la API de [SDL_Surface](https://wiki.libsdl.org/SDL_Surface) se mantiene prácticamente completa con respecto a SDL 1.2. Así que puedes cambiar tu surface de pantalla de esto:

	SDL_Surface *screen = SDL_SetVideoMode(640, 480, 32, 0);

A esto:

	// if all this hex scares you, check out SDL_PixelFormatEnumToMasks()!
	SDL_Surface *screen = SDL_CreateRGBSurface(0, 640, 480, 32,
										0x00FF0000,
										0x0000FF00,
										0x000000FF,
										0xFF000000);

	SDL_Texture *sdlTexture = SDL_CreateTexture(sdlRenderer,
											SDL_PIXELFORMAT_ARGB8888,
											SDL_TEXTUREACCESS_STREAMING,
											640, 480);

...y sigue bliteando cosas y modificando pixels como lo hacías antes, componiendo tu framebuffer final dentro de esta [SDL_Surface](https://wiki.libsdl.org/SDL_Surface). Una vez que esté listo para mostrarse en pantalla, puedes hacer como en el primer caso:

	SDL_UpdateTexture(sdlTexture, NULL, screen->pixels, screen->pitch);
	SDL_RenderClear(sdlRenderer);
	SDL_RenderCopy(sdlRenderer, sdlTexture, NULL, NULL);
	SDL_RenderPresent(sdlRenderer);

Ten en cuenta que crear texturas no es gratis: no llames [SDL_CreateTextureFromSurface()](https://wiki.libsdl.org/SDL_CreateTextureFromSurface) todos los frames. Crea una textura y una surface y actualiza la primera en base a la segunda.

La API de Render posee muchas otras características, algunas de las cuales probablemente puedan reemplazar código que antes escribías a mano: escalado, dibujado de líneas, etc. Si tus necesidades a la hora del renderizado no son muy complejas, probablemente te convenga dejar de modificar pixels a mano y mover todo al GPU. Tu programa funcionará más rápido y probablemente puedas simplificar notablemente tu código.

###Misceláneas sobre la Renderer API

Puedes realizar algunos efectos simples con la Renderer API sin necesidad de manipular directamente información de pixels. Algunos de los siguientes eran soportados en surfaces de SDL 1.2.

* Alpha de color: [SDL_Color](https://wiki.libsdl.org/SDL_Color) ahora tiene un cuarto componente dedicado al valor de alpha (transparencia). Probablemente tu código no utilizaba este valor en 1.2, en 2.0 deberías hacerlo.
* Alpha blending: utiliza [SDL_SetSurfaceAlphaMod](https://wiki.libsdl.org/SDL_SetSurfaceAlphaMod) y [SDL_SetTextureAlphaMod](https://wiki.libsdl.org/SDL_SetTextureAlphaMod) en vez de SDL_SetAlpha(). Es posible desactivar alpha blending en surfaces usando [SDL_SetSurfaceBlendMode()](https://wiki.libsdl.org/SDL_SetSurfaceBlendMode) y en texturas utilizando [SDL_SetTextureBlendMode()](https://wiki.libsdl.org/SDL_SetTextureBlendMode).
* Colorky: Cuando llames [SDL_SetColorKey()](https://wiki.libsdl.org/SDL_SetColorKey), deberás pasarle como parámetro SDL_TRUE en vez de SDL_SRCCOLORKEY.
* Modulación de color: Algunos renderer soportan alteraciones globales de color (srcC = srcC * color), puedes ver más detalles en [SDL_SetTextureColorMod()](https://wiki.libsdl.org/SDL_SetTextureColorMod).

###OpenGL

Si ya estabas utilizando OpenGL directamente, la migración es bastante simple. Necesitas cambiar tu llamada a SDL_SetVideoMode() por [SDL_CreateWindow()](https://wiki.libsdl.org/SDL_CreateWindow) seguida de [SDL_GL_CreateContext()](https://wiki.libsdl.org/SDL_GL_CreateContext. Luego cambia SDL_GL_SwapBuffers() por [SDL_GL_SwapWindow(window)](https://wiki.libsdl.org/SDL_GL_SwapWindow). Las llamadas de funciones de OpenGL no cambian.

Si utilizabas [SDL_GL_SetAttribute(SDL_GL_SWAP_CONTROL, x)](https://wiki.libsdl.org/SDL_GL_SetAttribute), necesitas hacer algunos cambios: ahora existe la función [SDL_GL_SetSwapInterval(x)](https://wiki.libsdl.org/SDL_GL_SetSwapInterval) que te permitirá modificar esto en un contexto GL existente.

Ahora SDL 2.0 es posible intercambiar entre modo ventana / pantalla completa sin perder el contexto de OpenGL (hurra!). Puedes controlar esto usando [SDL_SetWindowFullscreen()](https://wiki.libsdl.org/SDL_SetWindowFullscreen).

###Entrada

La buena noticia es que con SDL 2.0 es posible utilizar entrada Unicode. Las malas son que serán necesarios algunos cambios menores en tu aplicación.

En SDL 1.2, muchas aplicaciones que sólo soportaban Inglés de USA pero de todas maneras llamaban SDL_EnableUNICODE(1) porque les resultaba útil tener el caracter asociado con cada presión de tecla. Sin embargo, esto no funcionaba muy bien fuera del idioma Inglés y no funcionaba para nada en el caso de idiomas asiáticos.

Al parecer, soportar i18n es complicado.

SDL 2.0 cambia todo esto. SDL_EnableUNICODE() ya no existe más, junto con el campo unicode de [SDL_Keysym](https://wiki.libsdl.org/SDL_Keysym). Ya no recibes la información de caracter en los eventos [SDL_KEYDOWN](https://wiki.libsdl.org/SDL_EventType). Ahora, si usas [SDL_KEYDOWN](https://wiki.libsdl.org/SDL_EventType) debes tratarlo como si se tratara de un joystick con 101 botones. La entrada de texto viene por otro medio.

El nuevo evento es [SDL_TEXTINPUT](https://wiki.libsdl.org/SDL_EventType). Este evento se dispara siempre que el usuario ingrese texto. Ten en cuenta que este texto puede provenir del teclado o de algún tipo de IME (que es un método de ingresar texto complejo con varios caracteres). Este evento devuelte cadenas de texto completas, que puede ser de uno o más caracteres, siempre codificado en UTF-8.

Si sólo te interesa saber si el usuario presionó tal o cual tecla, puedes seguir utilizando [SDL_KEYDOWN](https://wiki.libsdl.org/SDL_EventType), con la diferencia que ahora el sistema se encuentra separado en dos partes con respecto a SDL 1.2: "keycodes" y "scancodes".

Los scancodes son independientes de la disposición del teclado. Puede pensar en ellos como "el usuario preionó la tecla Q" independientemente de que el teclado sea Americano, Europeo, Dvorak, etc. El scancode siempre indica el mismo caracter.

Los keycodes, por otra parte, sí dependen de la disposición del teclado. Piensa en ellos como "el usuario presionó la tecla Q en un teclado de idioma o disposición específico".

Por ejemplo, si presionas la tecla que se encuentra a dos posiciones de distancia de la tecla CAPS LOCK en un teclado QWERTY americano, se reportará como el scancode SDL_SCANCODE_S y el kaycode SDLK_S. La misma tecla en un teclado Dvorak será reportada como el scancode SDL_SCANCODE_S y el kaycode SDLK_O.

Presta atención a que ahora los scancodes ocupan 32 bits y usan un abanico amplio de números. Ya no existe más SDLK_LAST. Si tu programa chequeaba elementos SDLK_LAST para mapear entre teclas de SDL y lo que necesitara tu aplicación internamente, ten en cuenta que eso ya no es posible. Tendrás que usar una hash table, como por ejemplo std::map. Si estabas mapeando scancodes en vez de keycodes, ahora existe la constante SDL_NUM_SCANCODES para realizar chequeos de límites de array. En este momento, su valor es 512.

SDLMod ahora se llama [SDL_Keymod](https://wiki.libsdl.org/SDL_Keymod) y sus teclas "META" (la tecla "Windows") se llaman ahora teclas "GUI".

SDL_GetKeyState() ha sido renombrada a [SDL_GetKeyboardState()](https://wiki.libsdl.org/SDL_GetKeyboardState). El array retornado por dicha función ahora debe ser accedido utilizando valores SDL_SCANCODE_* (puedes ver [SDL_Scancode](https://wiki.libsdl.org/SDL_Scancode)) en vez de valores [SDL_Keysym](https://wiki.libsdl.org/SDL_Keysym).

Ahora veamos entrada de ratón.

El primer cambio, básicamente, es que la rueda del ratón ya no es tratada como un botón. Se trataba de un error histórico en la librería y lo corregimos en SDL 2.0. Ahora deberás chequear eventos [SDL_MOUSEWHEEL](https://wiki.libsdl.org/SDL_EventType). Hay soporte tanto para ruedas verticales como horizontales y algunas plataformas pueden reportar scroll con dos dedos en trackpads como eventos de rueda de ratón. Ya no recibirás eventos [SDL_BUTTONDOWN](https://wiki.libsdl.org/SDL_EventType) relacionados con la rueda y ahora los botones 4 y 5 son tratados como verdaderos eventos de botón de ratón.

Si tu juego requería que el mouse se desplazara indefinidamente en una dirección, por ejemplo para permitirle al jugador en un FPS girar por siempre sin que el ratón se detuviera al borde de la pantalla, probablemente ocultabas el cursor e interceptabas la entrada:

	SDL_ShowCursor(0);
	SDL_WM_GrabInput(SDL_GRAB_ON);

En SDL2, esto funciona un poco distinto. Sólo tienes que llamar:

	SDL_SetRelativeMouseMode(SDL_TRUE);

... y SDL se ocupa del resto.

###Eventos

[SDL_PushEvent()](https://wiki.libsdl.org/SDL_PushEvent) ahora retorna 1 en caso de éxito en vez de 0.

Las máscaras de evento ahora se especifican como rangos, entonces:

	SDL_PeepEvents(&event, 1, SDL_GETEVENT, SDL_EVENTMASK(SDL_MOUSEBUTTONDOWN));

se convierte en:

	SDL_PeepEvents(&event, 1, SDL_GETEVENT, SDL_MOUSEBUTTONDOWN, SDL_MOUSEBUTTONDOWN);

###Audio

Lo bueno es que, fuera de una sola excepción, SDL 2.0 es completamante compatible con SDL 1.2. Si quieres utilizar las nuevas funcionalidades adelante, pero probablemente puedas compilar y correr tu aplicación sin necesidad de utilizarlas.

Ahora, la excepción: El "callback" de audio ya NO comienza con su bufer inicializado completamente. En todos los casos debes escribir al bufer de manera completa. Si no tienes suficiente audio para hacerlo, tu callback deberá escribir silencio. Si no lo haces, escucharás el audio repetido o simple ruido. Si quieres volver al comportamiento anterior del bufer inicializado desde el vamos, deberás llamar SDL_memset(stream, 0, len) al comienzo de tu callback.

###Joysticks

Los eventos de Joystick ahora hacen referencia a un SDL_JoystickID. Esto es así porque SDL 2.0 ahora soporta joysticks que son conectados y desconectados en tiempo de ejecución de tu juego, haciendo inútil la lista de dispositivos usada por 1.2.

Para conseguir un SDL_JoystickID a partir de un SDL_Joystick* abierto, usa:

	SDL_JoystickID myID = SDL_JoystickInstanceID(myOpenedStick);

Y compara la propiedad which del evento contra myID. Si no utilizas la cola de eventos para joysticks, [SDL_JoystickGetAxis()](https://wiki.libsdl.org/SDL_JoystickGetAxis) y demás funcionan exactamente igual que en SDL 1.2.

Deberías revisar la nueva [API Game Controller](https://wiki.libsdl.org/CategoryGameController), es muy buena y probablemente te ahorre muchos dolores de cabeza que antes te producía SDL 1.2. Puedes encontrarla en SDL_gamecontroller.h. La API Game Controller se integra muy bien con el modo Big Picture de Steam: ofrece configuración automática para la mayoría de controladores y una buena interfaz visual si necesitas configurar algo a mano. En cualquier caso, Steam pasa el control de esta configuración a tu aplicación.

La API antigua de joysticks para Linux (/dev/input/js*) ha sido removida de SDL2. Ahora sólo se soporta la nueva API de eventos (/dev/input/event*) para joysticks. Normalmente estos eventos no son recibidos en cuentas de usuario comunes, así que incluso si se enchufan joysticks nuevos probablemente no sean detectados. Esto es algo que los usuarios finales tendrán que configurar por su cuenta.

###Hilos

SDL_KillThread() ya no existe más porque nunca fue lo suficientemente segura o confiable. La mejor manera de reemplazarla es setear un "flag" que le indique a un hilo que debe finalizar. El hilo deberá chequear este flag regularmente y luego el hilo "asesino" deberá llamar a [SDL_WaitThread()](https://wiki.libsdl.org/SDL_WaitThread) para limpiar todo.

Ahora [SDL_CreateThread()](https://wiki.libsdl.org/SDL_CreateThread) recibe un parámetro extra: un numbre para el hilo que puede ser utilizado por depuradores para identificarlo. Si no te interesa usar eso, puedes simplemente pasarle NULL.

###CDs de audio

La API de CD de SDL 1.2 fue removida completamente y no hay reemplazo. Lo más probable es que no liberes tu música en Audio CD a esta altura, si es que incluso liberas tu aplicación en CD. Puedes usar [Ogg Vorbis](http://www.vorbis.com/) o algún otro tipo de archivo de audio para tu música, la mayoría de los cuales son soportados por [SDL_Mixer](http://www.libsdl.org/projects/SDL_mixer/).

###Plataformas abandonadas

Removimos el soporte para unas cuantas plataformas antiguas, como OS/2 y Mac OS 9. Es más fácil enumerar las que todavía soportamos: Windows (XP y superior), Linux, Mac OS X, iOS y Android. Siguiendo la tradición de SDL, hay algunas otras plataformas externas que funcionan pero no son soportadas de manera completa, como Haiku y Sony PSP. En general, agregamos cualquier plataforma para la cual recibamos parches, pero nos pareció que ya era tiempo de decirle adiós a algunas de ellas al movernos a la nueva versión.

###Plataformas móviles

Hace bastante que existen ports no oficiales de SDL 1.2 para iOS y Android. Ahora SDL soporta esas plataformas directamente y la API 2.0 es más apropiada para las mismas. La mayor parte del contenido de este documento es válido en estas plataformas, pero existen algunas cosas que es necesario tener en cuenta.

Lo primero es que existen eventos que sólo aplican en dispositivos móviles o, mejor dicho, que aplican a la manera en la que los sistemas operativos móviles funcionan en el mundo post-iPhone. Originalmente tratábamos de mapear estos comportamientos a eventos ya existentes en SDL (como por ejemplo "tu aplicación pasa a segundo plano" emulado como una ventana perdiendo el foco), pero hay un problema importante: la mayoría de estos eventos requieren una respuesta inmediata y, si la aplicación no la provee, el SO cerrará tu aplicación inmediatamente.

Por eso hemos agregado nuevos eventos de SDL para ciertos detalles específicos de Android e iOS, pero tendrás que configurar un filtro de eventos para detectarlos tan pronto como el SO los reporte, porque si esperas hasta la próxima llamada a [SDL_PollEvent()](https://wiki.libsdl.org/SDL_PollEvent) ya será demasiado tarde.

Por ejemplo: tenemos SDL_APP_WILLENTERBACKGROUND, que es el equivalente de applicationWillResignActive() en iOS. Si dibujas algo luego de que ese evento se dispare, iOS cerrará tu aplicación. Por eso, necesitas interceptar esto de manera inmediata:

	int SDLCALL myEventFilter(void *userdata, SDL_Event * event)
	{
	    if (event->type == SDL_APP_WILLENTERBACKGROUND) {
	        // free up resources, DON'T DRAW ANY MORE until you're in the foreground again!
	    }
	    // etc
	    return 1;
	}

	// somewhere near startup...

	// this calls myEventFilter(data, event) as soon as event is generated.
	SDL_AddEventWatch(myEventFilter, data);

Además, ahora hay eventos de toque reales en vez de mapear esto al ratón. Puedes rastrear toques, varios dedos e incluso gestos complejos. Puedes ver la lista completa de funciones en SDL_touch.h y eventos [SDL_Finger*](https://wiki.libsdl.org/SDL_Finger) en SDL_events.h.

Hay algunas otras funciones pensadas para mobile, como [SDL_StartTextInput()](https://wiki.libsdl.org/SDL_StartTextInput) que mostrarán el teclado en pantalla. Úsalas.

Finalmente, hay algunas funciones específicas de Android e iOS para permitirte acceder a funcionalidades específicas de cada plataforma que no tendría mucho sentido agregar en una API general. Puedes ver la lista de estas funciones en SDL_system.h.

###RWops

Tanto [SDL_RWread()](https://wiki.libsdl.org/SDL_RWread) como [SDL_RWwrite()](https://wiki.libsdl.org/SDL_RWwrite) ahora retornan 0 en caso de error en vez de -1.

Si escribiste tu propia implementación de [SDL_RWops](https://wiki.libsdl.org/SDL_RWops), ten en cuenta que las declaraciones de las funciones cambiaron. Ahora utilizan Sint64 y size_t en vez de int para poder utilizar archivos grandes. En muchos casos, deberías poder actualizar las declaraciones y ya, pero si te habías chocado con estos problemas, seguro te alegrarás de saber que ya están solucionados. Las aplicaciones que invoquen a dichas funciones deben estar al tanto que los valores de retorno han cambiado.

También se agregó a RWops un método para calcular tamaño: [SDL_RWsize()](https://wiki.libsdl.org/SDL_RWsize). Esto permite a un RWops reportar el tamaño del "stream" sin obligar a la aplicación a realizar un "seek" hasta los cero bytes desde el final; en otras palabras, puedes reportar un tamaño para aquellas streams que no soporten seek. Si ni siquiera esto es posible, puedes retornar -1.

##Librerías complementarias

Las extensiones oficiales SDL_image, SDL_ttf, SDL_mixer y SDL_net ya tienen versiones correspondientes a SDL 2.0: SDL2_image, SDL2_ttf, SDL2_mixer y SDL2_net. Puedes que tengas que bajarlas desde los [repositorios de mercurial](http://hg.libsdl.org/) para tener los últimos arreglos. Ahora tu programa deberá linkear a, por ejemplo, SDL2_image en vez de SDL_image para poder compilar.

De ahora en más, estas librerías no darán más soporte a 1.2 y la compatibilidad con 1.2 probablemente desaparezca eventualmente con las nuevas versiones.

A partir de la versión 2.0.21 (Mayo 2010), es posible compilar [SDL_gfx](http://www.ferzkopp.net/joomla/content/view/19/14/) con SDL 2.0.

###Resumen de cosas renombradas o reemplazadas

* SDL_SetVideoMode(): usa [SDL_CreateWindow()](https://wiki.libsdl.org/SDL_CreateWindow) (junto con [SDL_CreateRenderer()](https://wiki.libsdl.org/SDL_CreateRenderer) si quieres hacer 2D clásico en vez de OpenGL)

* SDL_ListModes(): usa [SDL_GetDisplayMode()](https://wiki.libsdl.org/SDL_GetDisplayMode)/[SDL_GetNumDisplayModes()](https://wiki.libsdl.org/SDL_GetNumDisplayModes)

* SDL_UpdateRect()/SDL_Flip(): usa [SDL_RenderPresent()](https://wiki.libsdl.org/SDL_RenderPresent)

* SDL_Surface/2D rendering: todavía hay surfaces, pero es recomendable que uses [SDL_Textures](https://wiki.libsdl.org/SDL_Texture) con un renderer 2D acelerado ([SDL_CreateRenderer()](https://wiki.libsdl.org/SDL_CreateRenderer)) de ser posible.

* SDL_VideoInfo: usa [SDL_GetRendererInfo()](https://wiki.libsdl.org/SDL_GetRendererInfo)/[SDL_GetRenderDriverInfo()](https://wiki.libsdl.org/SDL_GetRenderDriverInfo)

* SDL_GetCurrentVideoDisplay(): usa [SDL_GetWindowDisplayIndex()](https://wiki.libsdl.org/SDL_GetWindowDisplayIndex)

* evento SDL_VIDEORESIZE: usa su equivalente [SDL_WINDOWEVENT_RESIZE](https://wiki.libsdl.org/SDL_WindowEvent)

###Lo demás

Hay una gran cantidad de cosas nuevas e interesantes en SDL 2.0 sobre las cuales 1.2 ni siquiera hubiera podido soñar. Aquí sólo tratamos de explicar aquello que pudieras tener que hacer para que tu programa compatible con 1.2 funcionara con 2.0, pero realmente es recomendable que explores la documentación. Probablemente ecuentres cosas que te hubiera gustado tener pero descartaste en su momento porque simplemente no existían. Por ejemplo: para cada juego que porté terminé con una función de mensaje de texto similar a esta:

	#if USING_SDL
	fprintf(stderr, "MSGBOX: %s\n%s\n", title, text);   // bueno, vaya.
	#endif

Ahora puedes usar [SDL_ShowSimpleMessageBox()](https://wiki.libsdl.org/SDL_ShowSimpleMessageBox). ¡De nada!

¡Si te salteaste alguna parte, puedes volver y chequear todas las funcionalidades nuevas en [el resumen](https://wiki.libsdl.org/MigrationGuide#Overview_of_new_features)!
