---
layout: default
title: Parte 4 · Comandos para consultar
nav_order: 5
---

# Parte 4 · Comandos para consultar
Todo lo de aquí abajo funciona igual desde la laptop o desde Termius en el celular.

## Lo básico de la terminal

| Comando | Qué hace |
| --- | --- |
| `ls -la` | Lista todo, incluidos los archivos ocultos, con permisos y tamaños |
| `cd carpeta` | Entra a una carpeta (`cd ..` sube una, `cd` sola te lleva a tu casa) |
| `pwd` | Dice en qué carpeta estás parado |
| `mkdir -p ruta/carpeta` | Crea una carpeta, y las intermedias si faltan |
| `cp -r origen destino` | Copia (la `-r` es para carpetas) |
| `cp README.md ~/Descargas/` | Copia un archivo a otra carpeta |
| `cp README.md README.bak` | Copia con otro nombre, o sea un respaldo |
| `cp -r Practica1 Practica1-respaldo` | Copia una carpeta entera |
| `mv origen destino` | Mueve, y también sirve para renombrar |
| `rm -r carpeta` | Borra, sin papelera y sin preguntar |
| `cat archivo` | Escupe el contenido de un archivo |
| `less archivo` | Lo abre para leerlo con calma (se sale con `q`) |
| `nano archivo` | Lo edita (`Ctrl + O` guarda, `Ctrl + X` sale) |
| `grep -r "texto" .` | Busca un texto dentro de todos los archivos de aquí abajo |
| `find . -name "*.pdf"` | Busca archivos por nombre |
| `chmod +x script.sh` | Le da permiso de ejecución a un archivo |
| `df -h` | Cuánto espacio le queda a tus discos |
| `du -sh carpeta` | Cuánto pesa una carpeta |
| `sudo dnf update -y` | Actualiza todo el sistema |

Tres costumbres que ahorran más tiempo que cualquier comando: `Tab` autocompleta nombres de archivos y carpetas a medias, `Ctrl + C` mata lo que esté corriendo, y `man` abre el manual de lo que sea. Por ejemplo, `man ls` te muestra todo lo que puedes hacer con ese comando.

## Borrar sin arrepentirte

En la terminal no hay papelera. Lo que borras con `rm` desaparece del disco en ese instante: no está en la papelera de KDE y no hay Ctrl+Z. Por eso `rm -rf` es el comando más peligroso de esta guía; si le pasas la ruta equivocada se lleva todo lo que haya debajo sin un solo aviso.

```bash
rm archivo.txt              # borra un archivo
rm -r Practica1-respaldo    # borra una carpeta con todo lo de adentro
rm -rf ~/ble.sh             # lo mismo, sin preguntar nada
```

La `-r` es para carpetas, igual que en `cp`. La `-f` fuerza: no pide confirmación ni se queja si algo no existe.

Dos costumbres que ahorran sustos: corre `ls` sobre la ruta antes de borrarla para confirmar que es la que crees, y mientras agarras confianza usa `rm -i`, que pregunta archivo por archivo.

Si prefieres poder arrepentirte, instala `trash-cli`. Manda las cosas a la papelera de KDE, de donde sí se recuperan:

```bash
sudo dnf install trash-cli -y
trash-put carpeta
```

## Borrar una carpeta pero conservar lo de adentro

No hay un comando que haga las dos cosas. Primero sacas el contenido y después borras la carpeta vacía.

```bash
mv Practica1/* .
rmdir Practica1
```

El `*` significa todo lo que hay dentro y el `.` es la carpeta donde estás parado, así que la primera línea sube todo un nivel. `rmdir` es el hermano seguro de `rm`: solo borra carpetas vacías y se niega si queda algo, que es justo lo que quieres aquí.

Ojo con un detalle que muerde: el `*` no toma los archivos ocultos, los que empiezan con punto. Si la carpeta tiene un `.git` o un `.env` se quedan atrás y `rmdir` te va a reclamar. Para esos casos:

```bash
mv Practica1/* Practica1/.* .
```

Se va a quejar de `.` y `..`, que no puede mover. Es normal. Confirma con `ls -la` que ya no quedó nada antes de borrar.

## Red

| Comando | Qué hace |
| --- | --- |
| `tailscale status` | Lista tus dispositivos, sus direcciones y cuáles están conectados |
| `tailscale ip -4` | Muestra la dirección de esta máquina |
| `sudo tailscale up` | Reconecta si te desconectaste |
| `sudo tailscale down` | Desconecta esta máquina de la red |
| `tailscale file cp archivo.pdf celular:` | Manda un archivo de la laptop al celular |
| `sudo tailscale file get ~/Descargas/` | Vacía la bandeja de Taildrop a mano |

## Control remoto

| Comando | Qué hace |
| --- | --- |
| `sudo systemctl suspend` | Duerme la laptop antes de guardarla |
| `sudo poweroff` | La apaga |
| `sudo reboot` | La reinicia |
| `top` | Qué está consumiendo CPU y RAM (se sale con `q`) |
| `w` | Quién está conectado y en qué terminal |
| `systemctl status taildrop-auto.service` | Revisa que el servicio de Taildrop siga vivo |

## tmux

| Comando | Qué hace |
| --- | --- |
| `tmux new -s trabajo` | Crea una sesión con nombre |
| `tmux attach -t trabajo` | Se engancha a una sesión existente |
| `tmux ls` | Lista las sesiones vivas |
| `tmux new-session -t trabajo -s ipad` | Se engancha desde un segundo dispositivo con su propio tamaño |
| `tmux kill-session -t trabajo` | Cierra una sesión a propósito |
| `Ctrl + b`, soltar, `d` | Sale sin matar nada |

## Lo que instalamos

| Comando | Qué hace |
| --- | --- |
| `Ctrl + R` | Abre el buscador de historial de Atuin |
| `atuin search "texto"` | Busca en el historial sin salir de la línea |
| `atuin stats` | Te dice qué comandos usas más |
| `z proyecto` | Salta a una carpeta frecuente escribiendo un pedazo del nombre (zoxide) |
| `zi` | Lo mismo pero eligiendo de una lista |
| `htop` | Monitor de sistema con colores (`sudo dnf install htop -y`) |

> **Detalle que confunde:** fzf también quiere quedarse con `Ctrl + R`, pero Atuin llega después y gana. Es lo que quieres, porque el de Atuin es mejor.

## Cuando algo no arranca

| Comando | Qué hace |
| --- | --- |
| `sudo systemctl daemon-reload` | Recarga los servicios después de editar uno |
| `sudo systemctl restart <servicio>` | Reinicia un servicio |
| `journalctl -u <servicio> -n 50` | Últimas 50 líneas del registro de un servicio |
| `exec bash` | Recarga la shell sin cerrar la ventana |
| `echo $XDG_SESSION_TYPE` | Dice si estás en Wayland o X11 |

---

Esta guía se va a seguir actualizando conforme aparezcan cosas nuevas o cambien los comandos, así que vale la pena volver al enlace en lugar de guardarte una copia. Aun así, no la tomes como la última palabra: los proyectos cambian de sintaxis entre versiones y lo que aquí funcionó puede quedarse viejo. Ante cualquier duda, revisa la documentación oficial de cada herramienta y compara con lo que diga la de tu distribución.

Fuentes: [Tailscale](https://tailscale.com/docs), [tmux](https://github.com/tmux/tmux/wiki), [Atuin](https://docs.atuin.sh), [zoxide](https://github.com/ajeetdsouza/zoxide), [fzf](https://github.com/junegunn/fzf), [ble.sh](https://github.com/akinomyoga/ble.sh).
