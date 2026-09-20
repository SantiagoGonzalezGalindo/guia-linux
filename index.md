---
layout: default
title: Inicio
nav_order: 1
---


*Última modificación: 20 de septiembre de 2026*

> **Aviso: lee la guía completa antes de ejecutar cualquier comando.** Varios pasos dependen de los anteriores y algunos modifican la configuración de arranque o de sesión del sistema. Ejecutarlos en desorden, a medias o copiando bloques sueltos puede dejarte sin entorno gráfico o sin poder iniciar sesión.

La guía va en cuatro partes. La primera monta la red privada entre la laptop y el celular, y sus pasos van en orden. La segunda mejora la terminal y es independiente: sirve aunque no instales nada de lo anterior. La tercera junta los errores y las confusiones con las que me topé, escritos por el mensaje que vas a ver en pantalla. La cuarta es una lista de comandos para tener a la mano.


## Qué vas a tener al terminar

Una red privada entre tu laptop y tu celular que funciona desde cualquier lugar: mandas archivos del celular a la laptop sin cables ni nube, y controlas la terminal de la laptop desde el celular aunque esté guardada en la mochila.

Esto se probó en Nobara Linux (derivado de Fedora, usa DNF5) con KDE Plasma sobre Wayland, más un iPhone. Si tu compañero trae Fedora, Bazzite o cualquier derivado, todo aplica igual. En Ubuntu o Debian solo cambia el gestor de paquetes. La parte del celular funciona igual en iPhone, iPad y Android.

No necesitas dejar la laptop prendida en casa ni tener IP fija ni abrir puertos en el módem. Tailscale resuelve eso solo.

Una advertencia antes de empezar: varios comandos que circulan por ahí para esto están mal escritos y te van a hacer perder media hora. Los de esta guía están verificados contra la documentación oficial de Tailscale. En la parte 3 hay una sección con los errores concretos para que no caigas en ellos.

## Qué sentido tiene juntar todo esto

Aquí hay dos cosas distintas que se tocan en un solo punto. Conviene entenderlo antes de instalar nada, porque así sabes qué te puedes saltar.

**Tailscale resuelve un problema de distancia.** Normalmente, para que tu celular hable con tu laptop tienen que estar en la misma red, o tú tienes que abrir puertos en el módem y dejar tu máquina expuesta a internet. Tailscale quita esa disyuntiva: le da a cada equipo una dirección fija privada y los conecta cifrado, estén donde estén. De ahí salen las tres cosas de la parte 1: mandarte archivos sin cables ni nube de por medio, entrar a la terminal de tu laptop desde el celular, y llamar a tus máquinas por su nombre en lugar de por un número. Más adelante, si montas un servidor de archivos o quieres enseñarle un proyecto a un compañero sin subirlo a ningún lado, la tubería ya está puesta.

**Lo de la parte 2 resuelve un problema de fricción.** tmux, Atuin, zoxide, fzf y ble.sh no te dejan hacer nada que antes fuera imposible; te quitan segundos y esfuerzo mental de cosas que haces cien veces al día. tmux hace que tu terminal deje de morir cuando cierras la ventana, aunque eso no significa que se guarde para siempre: la sesión vive mientras la laptop siga encendida y desaparece al apagarla o reiniciarla. Atuin convierte tu historial en algo que se puede consultar en lugar de una lista ciega. zoxide te quita el recordar rutas y fzf el recordar nombres. ble.sh te muestra lo que probablemente ibas a escribir.

**El punto donde se cruzan los dos grupos es tmux.** Sin él, entrar por SSH desde el celular te da una terminal nueva y vacía cada vez: sirve para apagar la laptop o revisar algo rápido, pero no para trabajar. Con él hay una sesión permanente a la que te asomas desde donde sea. Dejas algo compilando en la biblioteca, guardas la laptop, y desde el celular en el camión ves cómo va. Ese es el único caso donde la suma vale más que las partes.

El resto no se combina, y eso no es un defecto sino la razón por la que puedes instalarlas con confianza. Cada una se justifica sola: Atuin vale la pena aunque nunca uses zoxide, y fzf te sirve aunque jamás instales ble.sh. Como no dependen unas de otras, puedes agregarlas de una en una, quedarte con las que te acomoden y quitar cualquiera sin romper las demás. Lo decimos aquí para que no las instales todas de golpe pensando que solo funcionan juntas, porque entonces no vas a saber cuál te está ayudando y cuál te estorba. De hecho lo único que sí se cruza entre ellas es cuando dos quieren el mismo atajo de teclado, y eso queda resuelto en los pasos correspondientes.

## Cómo leer esta guía

Si es la primera vez que trabajas en la terminal, léete esto antes. Te va a ahorrar la mayoría de los tropiezos.

**Dónde se escriben los comandos.** En KDE la terminal se llama Konsole y la abres desde el menú de aplicaciones o con `Ctrl + Alt + T`. Todo lo que está en los recuadros grises de esta guía va ahí. Se pega con `Ctrl + Shift + V`, no con `Ctrl + V`, que en la terminal significa otra cosa.

**Las palabras que tienes que sustituir.** Cuando veas `cd carpeta`, `rm archivo` o `cp origen destino`, esas palabras son huecos, no comandos literales. Van reemplazadas por el nombre real de tu carpeta o tu archivo. Si escribes `man comando` tal cual, Linux te va a decir que no existe ninguna entrada para "comando", porque tomó la palabra en serio.

**Los símbolos que van a salir seguido.** El `~` es tu carpeta personal, o sea `/home/tu-usuario`; `~/Descargas` significa la carpeta Descargas que está dentro de ella. El `$USER` se reemplaza solo por tu nombre de usuario, así que ese sí lo copias tal cual. Y todo lo que va después de un `#` dentro de un recuadro es un comentario para ti, no forma parte del comando.

**Cuando un comando pide contraseña.** Los que empiezan con `sudo` corren como administrador y te la van a pedir. Al escribirla no se ve nada en pantalla, ni asteriscos ni puntos: es normal, escríbela completa y presiona Enter.

**Cuando algo no funciona.** Si te dice `instrucción no encontrada`, revisa primero que no tengas un error de dedo, porque es lo más común. Si está bien escrito, casi siempre es que falta instalar el programa o que la terminal sigue siendo la de antes; `exec bash` la recarga. La parte 3 tiene los casos concretos con los que nos topamos.

Y si nada de eso lo resuelve, pregúntale a la IA que uses. El truco está en cómo le pasas el error: copia de la terminal el bloque completo, incluyendo la línea que escribiste tú, no nada más el mensaje de error. Esa línea es la que le dice qué intentabas hacer.

```bash
usuario@laptop:~$ tailsca
bash: tailsca: instrucción no encontrada...
```

Con eso cualquier IA ve de inmediato que el comando está mal escrito. Si le mandas solo "instrucción no encontrada" va a tener que adivinar, y ahí es donde empieza a inventar soluciones. Aun así, revisa lo que te conteste antes de ejecutarlo: más de un comando de los que nos dictaron durante esta configuración simplemente no existía.
