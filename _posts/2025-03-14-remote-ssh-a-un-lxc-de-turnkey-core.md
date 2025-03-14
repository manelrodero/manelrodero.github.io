---
layout : post
blog-width: true
title: 'Remote SSH a un LXC de TurnKey Core'
date: '2025-03-14 18:24:03'
#last-updated: '2025-03-14 18:24:03'
published: true
tags:
- Linux
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2025-03-14_cover.png"
thumbnail-img: ""
---

Cuando se intenta abrir una conexión [**Remote-SSH**](https://code.visualstudio.com/docs/remote/ssh){:target=_blank} desde [Visual Studio Code](https://code.visualstudio.com/){:target=_blank} a un contenedor LXC basado en TurnKey Core aparece el siguiente mensaje de error: "_Setting up SSH Host hostname: Waiting for port forwarding to be ready_".

![Remote-SSH Port Forwarding][2]

El problema reside en la configuración especificada en el fichero `/etc/ssh/sshd_config.d/turnkey.conf`:

```plaintext
# SSH hardening recommended by lynis
X11Forwarding no
AllowTcpForwarding no
ClientAliveCountMax 2
MaxAuthTries 3
MaxSessions 2
```

Como se puede observar, esta opción está deshabilitada. Habría que cambiar el valor de `AllowTcpForwarding` a `yes` y reiniciar el servicio mediante el comando `systemctl restart sshd.service`.

### Historial de cambios

* **2025-03-14**: Documento inicial

[2]: /assets/img/blog/2025-03-14_image_2.png "Remote-SSH Port Forwarding"
