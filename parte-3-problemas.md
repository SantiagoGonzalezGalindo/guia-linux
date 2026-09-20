---
layout: default
title: Parte 3 · Cuando algo sale mal
nav_order: 4
---

# Parte 3 · Cuando algo sale mal
Todos estos salieron de intentarlo en vivo. Varios vienen de comandos que una IA inventó con mucha seguridad y que no existen.

**No reinicies `systemd-logind` con la sesión gráfica abierta.** El comando `sudo systemctl restart systemd-logind` deja el escritorio en un bucle de inicio de sesión: metes la contraseña, se apagan los monitores y regresas a la pantalla de login. Si tocas algo en `logind.conf`, reinicia la computadora completa en lugar del servicio. Y si ya caíste en el bucle, se sale con `Ctrl + Alt + F3` para entrar a una terminal de texto, o directo por SSH desde el celular.

**Cuidado con los comandos que una IA te dicta.** Además de la URL del instalador sin `/install.sh`, en nuestra sesión aparecieron `curl ... | /install.sh`, que busca un archivo en la raíz del sistema, y `--add-repo`, que DNF5 ya no acepta. Cuando un comando falle dos veces de la misma forma, busca la documentación oficial en lugar de pedir otra variante.

## Confusiones de la primera noche

Todas estas me pasaron a mí armando esto. Si te topas con alguna, aquí está la explicación.

**Se murió la sesión de tmux y se salieron los dos dispositivos.** No fue que uno arrastrara al otro: presionaste `Ctrl + d`, que cierra la shell y mata la sesión completa. El detach son dos pulsaciones separadas, `Ctrl + b`, soltar, `d`.

**Instalaste algo y el comando no existe.** Casi siempre es que la shell sigue siendo la de antes. Corre `exec bash` y vuelve a probar. Y si estás dentro de tmux, eso solo arregla el panel donde lo corriste; los demás siguen con la shell vieja.

**`z: instrucción no encontrada` aunque instalaste zoxide.** `z` no es un programa, es una función que hay que inyectar en la shell con `eval "$(zoxide init bash)"` en el `~/.bashrc`. Lo mismo aplica a fzf con `eval "$(fzf --bash)"`.

**`zoxide: no match found` con una carpeta que sí existe.** Zoxide solo conoce carpetas donde ya estuviste desde que lo instalaste; no escanea el disco. Entra una vez con `cd` y a partir de ahí ya te lleva.

**Atuin te ofrece crear una cuenta para sincronizar.** Dile que no. Entrar por SSH desde el celular no es otra máquina: los comandos corren en la laptop y el historial ya es el mismo. La sincronización solo sirve el día que tengas un segundo equipo con su propio Linux.

**Clonaste algo y apareció una carpeta rara dentro de tu proyecto.** `git clone` descarga donde estés parado, no pregunta. Muévete a `~` antes de clonar, o borra la carpeta sobrante después; si ya corriste `make install`, la instalación no depende de ella.

**El comando no se encuentra y lo escribiste bien.** Revísalo letra por letra antes de buscar en internet. `autin` por `atuin` me costó un rato.

**Los archivos que llegan por Taildrop no se dejan borrar.** Es el servicio corriendo como root. Recuperas el control con `sudo chown -R $USER:$USER ~/Descargas/Taildrop/` y evitas que se repita con la línea `User=` en el servicio, como está en el paso 4.
