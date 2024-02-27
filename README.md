# Instalación de HikCentral Professional 2.5 con Hyper-V en Windows 10 dentro de ProxmoxVE 8.1.1

## [Corroborar la opción de *Nested Hypervisor/Virtualization*](https://dannyda.com/2020/09/21/how-to-enable-nested-hypervisor-virtualization-on-proxmox-ve-pve/)

> **Importante**: *Microsoft Hyper-V* solo funciona con Intel CPUs recientes. 
Esta guía se rige a dichos CPUs.

Para comprobar si la virtualización aninada está habilitada, abra la terminal/SSH/Shell, y ejecute el siguiente comando:
```shell
cat /sys/module/kvm_intel/parameters/nested
```

Si la respuesta es `N`, esta **desactivada**. Para habilitarlo:
```shell
echo "options kvm-intel nested=Y" > /etc/modprobe.d/kvm-intel.conf
modprobe -r kvm_intel
modprobe kvm_intel 
```
- **Nota**: Si lo ultimo tira error, solo reinicia el PVE host.

Luego vuelve a chequear con:
```shell
cat /sys/module/kvm_intel/parameters/nested
```
Debería mostrar la letra `Y`.

### Conocer las especificaciones básicas del servidor.

En el PVE host, abra la terminal/SSH/Shell y ejecute el siguiente comando para saber los *cores*: `nproc`.  
Si quieres mas información usa el comando: `cat /proc/cpuinfo`.

Para conocer la memoria usa el comando: `free` para ver el total y la memoria dispobile, o `free -h` para leerlo en valores entendibles.

Por ultimo para ver la capacidad de el o los discos ejecuta el comando: `lsblk`, o si eres basico usa `df -h`.

### Subida de archivos al PVE host.

Descargar las ISO respectiva de Windows 10.  
Descarga la ISO más estable de VirtIO drivers para Windows desde [acá](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso).  
Empaquetar el instalador de HikCentral descargado de [aquí](https://www.hikvision.com/content/dam/hikvision/en/support/download/vms/hcp-2-5-0/HikCentral-Professional_Full-Pack_V2.5.0.202311082120_Win_x64_Installer.exe) en un ISO.

Para subir las ISOs, ir a la base de datos local del proxmox. clic en ISO Images y darle al botón Upload.  
Luego selecciona el archivo en cuestión y clic en el botón azul de Upload. Espera que en la nueva ventana aparezca `TASK OK` para cerrarla.

## Configurar la nueva máquina virtual.

Siguiendo los pasos de este [video](https://www.youtube.com/watch?v=anmV3nS6i20), para crear la maquina virtual: 
1. - Se da un nombre y el pestaña OS, seleccionamos la ISO del sistema.   
    - Se ajusta el *Guest OS* a `Windows` y la versión a `10/2016/2019`.  
    - Se marca la opción de VirtIO drivers y se seleccionar los mismos.
2. - En la pestaña System, Machine es cambiado a `qt35`.
    - El BIOS seleccionado es `OVMF(UEFI)`, en mi caso he usado el local-lvm para el disco EFI.
    - El controlador SCSI lo puedes usar con VirtIO SCSI o la versión single, la cual he usado.
3. En la pestaña Disks, se selecciona el dispositivo SCSI, y se cambia el tamaño a 650GB para la base de datos.
4. - En la pestañas CPUs, se cambia los cores a mínimo 8, y cambia el Type a `host`.
    - Se activa las funciones avanzadas con el checkbox junto botones de abajo.
    - Buscar que `hv-evmcs` este `ON`. He activado además `spec-ctrl`.
5. Cambiar la memoria a 8GB (8192) como mínimo, y recomendable 16GB.
6.  En la pestaña Network, aunque se debería usar VirtIO driver, no ha funcionado, así que puede ir con Intel E1000.
7. Acepta y finaliza la configuración. 

## Instalación de la máquina virtual y activación de Hyper-V

- Iniciar la máquina virtual e iniciar el instalador de Windows. 
- Cuando se encuentre en la selección del disco, darle clic al botón con el disco, y luego a Browse.
- Busque en el CD de VirtIO > vioscsi >  w10 > amd64 y seleccionarlo. 
- Cargue el driver y prosiga con la instalación.

Una vez instalado el sistema, abrir Powershell como administrador, y escribir el comando:
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```
Al terminar la instalación va a pedir reiniciar, escriba `Y` para aceptar la actualización.

Para chequear que este todo instalado, siga estos [pasos](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) y su máquina virtual debe lucir exactamente igual a la imagen con todos checkboxes completos en Hyper-V.

Finalmente, montar en la máquina virtual un CD con la ISO de HikCentral e instalarlo.  
No debe haber errores al visualizar el cliente web.
