---
layout : post
blog-width: true
title: 'Habilitar CUDA para Immich en Unraid'
date: '2025-09-21 18:46:48'
last-updated: '2025-09-22 18:46:48'
published: true
tags:
- Unraid
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-09-21_cover.png"
thumbnail-img: ""
---

Recientemente he revisado mi [instalación de Immich en Unraid](instalar-immich-en-unraid) para utilizar las mismas imágenes que la instalación oficial mediante [Docker Compose](https://immich.app/docs/install/docker-compose){:target="_blank"}.

Ahora llega el momento de usar la **GPU** que tiene el servidor Unraid para acelerar las tareas de [**Machine Learning**](https://immich.app/docs/features/ml-hardware-acceleration){:target="_blank"} (Smart Search, Facial Recognition, etc.) para reducir la carga de CPU.

# GeForce GTX 1650

Mi servidor tiene una **GeForce GTX 1650** que es totalmente compatible con [**CUDA**](https://docs.nvidia.com/cuda/){:target="_blank"}:

* Tools &rarr; System Devices &rarr; PCI Devices and IOMMU Groups
  * IOMMU group 1:
    * [8086:1901] 00:01.0 PCI bridge: Intel Corporation 6th-10th Gen Core Processor PCIe Controller (x16) (rev 03)
    * [10de:1f82] 01:00.0 VGA compatible controller: **NVIDIA Corporation TU117 [GeForce GTX 1650] (rev a1)**
    * [10de:10fa] 01:00.1 Audio device: NVIDIA Corporation Device 10fa (rev a1)

Esta tarjeta tiene una [**Compute Capability**](https://developer.nvidia.com/cuda-gpus){:target="_blank"} de **7.5**, compatible con CUDA 10.1 y posteriores, que soporta:

* Tensor cores (aunque limitados en esta gama)
* FP16 y INT8 acelerado
* CUDA Unified Memory
* Acceso a bibliotecas como cuDNN, TensorRT, PyTorch, etc.

## Nvidia-Driver

![Plugin Nvidia-Driver][2]

Unraid tiene un **plugin oficial** que instala todos los módulos y dependencias necesarios para poder utilizar las tarjetas gráficas de NVIDIA en contenedores Docker.

Este plugin, llamado `Nvidia-Driver`, se instala desde las aplicaciones de la comunidad:

* Apps &rarr; Search &rarr; Nvidia-Driver &rarr; Install

El plugin descarga e instala la última versión del paquete de NVIDIA disponible (a día de hoy, 21 de septiembre de 2025, es la **v580.82.09**).

Durante este proceso, que tarda poco menos de un minuto, no hay que cerrar la ventana donde se muestra el progreso de la instalación para evitar que la instalación quede a medias.

Para poder utilizar este driver dentro de contenedores Docker, hay que reiniciar Docker de la siguiente manera:

* Docker &rarr; Stop All
* Settings &rarr; Docker
  * Enable Docker: **No** &rarr; Apply
  * Enable Docker: **Yes** &rarr; Apply

El estado de la tarjeta gráfica se puede ver desde una _shell_ de Unraid usando el comando `nvidia-smi`:

![nvidia-smi][1]

El repositorio del plugin está en GitHub: [https://github.com/unraid/unraid-nvidia-driver](https://github.com/unraid/unraid-nvidia-driver){:target="_blank"}.

## NVTOP

![Plugin NVTOP][3]

Unraid también tiene un **plugin** que instala la [herramienta `nvtop`](https://itsfoss.gitlab.io/post/nvtop--awesome-linux-task-monitor-for-nvidia-amd--intel-gpus/){:target="_blank"} que permite monitorizar la tarjeta gráfica en tiempo real mediante una interfaz similar a `htop`.

Este plugin, llamado `NVTOP`, se instala desde las aplicaciones de la comunidad:

* Apps &rarr; Search &rarr; NVTOP &rarr; Install

El plugin descarga e instala la última versión de la herramienta disponible (a día de hoy, 22 de septiembre de 2025, es la **v3.2.0**).

![nvtop][4]

El repositorio del plugin está en GitHub: [https://github.com/ich777/unraid-nvtop](https://github.com/ich777/unraid-nvtop){:target="_blank"}.

# Machine Learning con GPU (CUDA)

Para poder [usar la GPU para acelerar las tareas de Machine Learning](https://immich.app/docs/features/ml-hardware-acceleration){:target="_blank"} es necesario cambiar la imagen del contenedor de ML de Immich por una que tenga soporte para CUDA (`ghcr.io/immich-app/immich-machine-learning:v1.142.1-cuda`).

Además, hay que exponer la tarjeta gráfica dentro del contenedor utilizando las variables `NVIDIA_VISIBLE_DEVICES` y `NVIDIA_DRIVER_CAPABILITIES` tal como indica la documentación del [**NVIDIA Container Toolkit**](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/docker-specialized.html){:target="_blank"}.

El contenido del fichero `my-immich-machine-learning-cuda.xml` es el siguiente:

```xml
<?xml version="1.0"?>
<Container version="2">
    <Name>immich-machine-learning-cuda</Name>
    <Repository>ghcr.io/immich-app/immich-machine-learning:v1.142.1-cuda</Repository>
    <Network>immich-net</Network>
    <Shell>bash</Shell>
    <Privileged>false</Privileged>
    <Icon>https://raw.githubusercontent.com/homarr-labs/dashboard-icons/refs/heads/main/png/immich.png</Icon>
    <ExtraParams>--runtime=nvidia --hostname immich-machine-learning --dns=1.1.1.1</ExtraParams>
    <Config Name="IMMICH_LOG_LEVEL" Target="IMMICH_LOG_LEVEL" Default="log" Mode="" Description="Log level (verbose, debug, log, warn, error)" Type="Variable" Display="advanced" Required="false" Mask="false">log</Config>
    <Config Name="Machine Learning model cache" Target="/cache" Default="/mnt/user/appdata/immich-machine-learning/cache" Mode="rw" Description="Directory where models are downloaded" Type="Path" Display="advanced" Required="true" Mask="false">/mnt/user/appdata/immich-machine-learning/cache</Config>
    <Config Name="TZ" Target="TZ" Default="Europe/Madrid" Mode="" Description="Container time zone" Type="Variable" Display="always" Required="false" Mask="false">Europe/Madrid</Config>
    <Config Name="NVIDIA_VISIBLE_DEVICES" Target="NVIDIA_VISIBLE_DEVICES" Default="all" Mode="" Description="Which GPU devices to expose (e.g., all or 0,1)" Type="Variable" Display="advanced" Required="false" Mask="false">all</Config>
    <Config Name="NVIDIA_DRIVER_CAPABILITIES" Target="NVIDIA_DRIVER_CAPABILITIES" Default="all" Mode="" Description="GPU capabilities" Type="Variable" Display="advanced" Required="false" Mask="false">all</Config>    
</Container>
```

## Proveedores de inferencia

Se puede comprobar qué **proveedores de inferencia** hay disponibles en el nuevo contenedor usando el siguiente comando:

```bash
docker exec -it immich-machine-learning-cuda python3 -c "import onnxruntime; print(onnxruntime.get_available_providers())"
```

Cuando se utiliza la imagen con soporte para CUDA la salida es la siguiente:

```plaintext
['TensorrtExecutionProvider', 'CUDAExecutionProvider', 'CPUExecutionProvider']
```

Es decir, se tiene acceso a completo a:

* `CUDAExecutionProvider`: soporte activo para inferencia acelerada por GPU
* `TensorrtExecutionProvider`: soporte para TensorRT (aún más rápido en algunos modelos)
* `CPUExecutionProvider`: _fallback_ por CPU disponible

Cuando se utiliza la imagen sin soporte para CUDA la salida era la siguiente:

```plaintext
['AzureExecutionProvider', 'CPUExecutionProvider']
```

## Confirmar el uso de la GPU

La forma más sencilla de confirmar que se está utilizando la GPU es ejecutar el comando `nvtop` mientras se ejecuta una tarea de Machine Learning como puede ser **Face Detection**.

* `Administration` &rarr; `Jobs` &rarr; `Face Detection` &rarr; **Reset**

Si todo funciona correctamente, se debería ver cómo se incrementa el uso de la GPU y la memoria de la tarjeta gráfica:

![nvtop ML GPU][5]

### Historial de cambios

* **2025-09-21**: Documento inicial
* **2025-09-22**: Confirmación del uso de CUDA (nvtop)

[1]: /assets/img/blog/2025-09-21_image_1.png "nvidia-smi"
[2]: /assets/img/blog/2025-09-21_image_2.png "Plugin Nvidia-Driver"
[3]: /assets/img/blog/2025-09-21_image_3.png "Plugin NVTOP"
[4]: /assets/img/blog/2025-09-21_image_4.png "nvtop"
[5]: /assets/img/blog/2025-09-21_image_5.png "nvtop ML GPU"
