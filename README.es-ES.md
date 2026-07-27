# Lenovo ThinkPad X280 - Configuración de OpenCore

<img align="right" src="https://i.imgur.com/zbcphj3.jpg" alt="macOS Ventura running on the X280" width="350">

[![macOS](https://img.shields.io/badge/macOS-Monterey-brightgreen.svg)](https://developer.apple.com/documentation/macos-release-notes)
[![OpenCore](https://img.shields.io/badge/OpenCore-0.8.5-blue)](https://github.com/acidanthera/OpenCorePkg)
[![License](https://img.shields.io/badge/license-MIT-purple)](/LICENSE)

<p align="center">
   <strong>Estado: Sin mantenimiento (y funciona a medias)</strong>
   <br />
   <strong>Versión: </strong>1.0
   <br />
</p>
</br>

## ⚠️ Descargo de responsabilidad
**Esta guía se basa fuertemente en [la guía de valnoxy para el T480.](https://github.com/valnoxy/t480-oc) Esto es posible debido a las similitudes de hardware. Ten en cuenta que esto aún no es perfecto y TENDRÁS problemas.**

Esta guía es solo para el Lenovo ThinkPad X280. NO soy responsable de cualquier daño que causes a tu dispositivo. Esta guía se proporciona "tal cual" y todos los pasos realizados son bajo tu propio riesgo.

> Los parches ACPI y el estilo de este README son de [EETagent](https://github.com/EETagent/T480-OpenCore-Hackintosh).

## Introducción

<details>
<summary><strong>💻 Mi Hardware</strong></summary>
<br>
Esta es la configuración de mi X280; sin embargo, esta configuración <strong>debería seguir funcionando</strong> con todas las variantes de X280.

> **Nota** Verifica el modelo de tu tarjeta WiFi y Bluetooth. Las tarjetas Intel deberían ser compatibles con itlwm (o AirportItlwm). Si tu tarjeta es de otro fabricante, por favor verifica si tu tarjeta es compatible con macOS.

| Categoría | Componente                            |
| --------- | ------------------------------------ |
| CPU       | Intel Core i5-8250U                  |
| GPU       | Intel UHD Graphics 620               |
| SSD       | Intel SSDPEKKF256G8L M.2 NVMe SSD    |
| Memoria    | 8GB DDR4 2400Mhz                     |
| WiFi      | Intel Dual Band Wireless-AC 8265     |

</details>  

&nbsp;

## Instalación

<details>  
<summary><strong>📝 Requisitos</strong></summary>
</br>

Debes tener los siguientes elementos:
- un X280 que funcione (obviamente);
- acceso a una máquina con Windows que funcione y tenga Python 3 instalado;
- una memoria USB de más de 4 GB (recuerda que durante la preparación formatearemos la unidad flash para crear el medio de instalación, lo que **borrará todos los datos que contenga actualmente**);
- una conexión a Internet (se recomienda vía Ethernet mediante adaptador, pero el Wi-Fi debería funcionar bien);
- unas cuantas horas para solucionar problemas. **¡Si apresuras la instalación, es seguro que arruinarás algo!**

</details>

<details>  
<summary><strong>⚙️ Preparación</strong></summary>
</br>

### Crear el medio de instalación

En primer lugar, necesitarás un instalador de macOS. Usaremos [macrecovery](https://github.com/acidanthera/OpenCorePkg) para descargar y crear la unidad USB.

Con macrecovery, el proceso es el siguiente:
- Descarga [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg) como ZIP.
- Extrae el archivo OpenCorePkg-master.zip.
- Abre ```cmd.exe``` con privilegios de Administrador y cambia el directorio a OpenCorePkg-master\Utilities\macrecovery.
- Ingresa el siguiente comando para descargar macOS 12:
```
python macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000 download
```
- Una vez que la descarga haya tenido éxito, escribe ```diskpart``` y espera hasta que veas ```DISKPART>```

- Conecta tu pendrive y escribe ```list disk``` para ver el ID de tu disco.

- Selecciona tu pendrive escribiendo ```select disk <diskid>```. **¡Asegúrate de haber seleccionado el dispositivo correcto! Usa ```detail disk``` para verificar.**

- Ahora, limpia el pendrive y conviértelo a GPT; primero, escribe ```clean``` y luego ```convert gpt```.

>  **Nota**: Si ocurrió un error, intenta convertir de nuevo escribiendo ```convert gpt```.

- **Si tu unidad es de más de 32GB, tendrás problemas con el siguiente paso** - DiskPart solo puede manejar el formato FAT32 para unidades de hasta 32GB, por lo que para crear una partición FAT32 necesitarás una herramienta externa. Yo personalmente recomiendo [minitool's partition wizard](https://cdn2.minitool.com/?p=pw&e=pwfree-64bit-portable). Un tutorial para esto se agregará más adelante. Si sigues este camino, sáltate el siguiente paso (y **SOLAMENTE** el siguiente paso).

- Después de que el pendrive esté limpio y convertido, crea una nueva partición para el instalador y EFI; escribe ```create partition primary```, luego selecciona la nueva partición con ```select partition 1``` y formatéala ```format fs=fat32 quick```.

- Finalmente, monta tu pendrive escribiendo ```assign```

- Ahora, cierra el Símbolo del sistema y copia la carpeta ```com.apple.recovery.boot``` (con su contenido) desde ```OpenCorePkg-master\Utilities\macrecovery``` hacia la raíz del pendrive.

Después de crear el medio de instalación, necesitas hacer que la unidad USB sea arrancable.

### Configurar e instalar OpenCore
Descarga la carpeta EFI de este repositorio, descarga el repo como .zip y arrastra la carpeta ```EFI``` a la raíz de tu pendrive.

#### GenSMBIOS
Para configurar tu SMBIOS, necesitas usar [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS). La herramienta generará automáticamente un número de serie falso, UUID y MLB para ti. **¡Este paso es esencial para que iMessage funcione, así que no lo omitas!**

- Descarga GenSMBIOS como ZIP y luego extráelo.
- Inicia GenSMBIOS.bat y usa la opción ```1``` para descargar MacSerial.
- Elige la opción ```2``` para seleccionar la ruta del archivo config.plist. Se encontrará en la carpeta ```EFI -> OC``` de tu pendrive.
- Elige la opción ```3``` e ingresa ```MacBookPro15,2``` como el tipo de máquina.
- Presiona ```Q``` para salir. Tu configuración ahora debería contener los seriales requeridos.

#### Ingresar el valor ROM adecuado
Después de agregar los seriales a tu config.plist, debes agregar la dirección MAC de la computadora al archivo config.plist. **Este paso también es esencial para que iMessage funcione, así que no lo omitas.** Necesitamos un editor de ```.plist``` para escribir la dirección MAC en el archivo config.plist. Recomiendo [ProperTree](https://github.com/corpnewt/ProperTree) ya que funciona en Windows. Debes cambiar el valor de la dirección MAC en el config.plist en:

```PlatformInfo -> Generic -> ROM```

Borra el valor genérico ```123456789012``` e ingresa tu dirección MAC en el campo, sin dos puntos. Puedes obtener esta dirección en [Windows](https://kb.netgear.com/1005/How-do-I-find-my-device-s-MAC-address) y [Linux](https://cets.seas.upenn.edu/answers/find-mac-address.html) siguiendo los tutoriales enlazados. Guarda el archivo ```.plist``` presionando ```Ctrl+S```.

#### Distribución de teclado e idioma predeterminados
La distribución de teclado y el idioma predeterminados son ```Polish``` (Polaco). Para cambiar el idioma, edita el valor de ```NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> prev-lang:kbd``` al valor de tu idioma. Esta parte es un poco complicada; si tu valor contiene un guion bajo "```_```", reemplázalo por un guion medio "```-```". El valor para inglés sería ```en-US:0```. Puedes encontrar una lista de todos los valores de idioma [aquí](https://github.com/acidanthera/OpenCorePkg/blob/master/Utilities/AppleKeyboardLayouts/AppleKeyboardLayouts.txt). Por ejemplo, si eligieras portugués, el valor en este archivo es ```[10] pt_PT - Portuguese```; por lo tanto, el valor que debes poner en tu ```.plist``` sería ```pt_PT:10```.

### Instalar OpenCore
Si seguiste todos los pasos correctamente y copiaste la carpeta ```EFI``` anteriormente, OpenCore ya debería estar instalado y listo para arrancar.

</details>

<details>  
<summary><strong>🚚 Instalación</strong></summary>
</br>

### Preparar la BIOS
La BIOS **debe** estar configurada correctamente antes de instalar macOS.
En el menú de Security, establece los siguientes ajustes:

-  `Security > Security Chip`: debe estar en **Disabled**
-  `Memory Protection > Execution Prevention`: debe estar en **Enabled**
-  `Virtualization > Intel Virtualization Technology`: debe estar en **Enabled**
-  `Virtualization > Intel VT-d Feature`: debe estar en **Enabled**
-  `Anti-Theft > Computrace -> Current Setting`: debe estar en **Disabled**
-  `Secure Boot > Secure Boot`: debe estar en **Disabled**
-  `Intel SGX -> Intel SGX Control`: debe estar en **Disabled**
-  `Device Guard`: debe estar en **Disabled**

En el menú de Startup, establece las siguientes opciones:

-  `UEFI/Legacy Boot`: **UEFI Only**
-  `CSM Support`: **No**

En el menú de Thunderbolt, establece las siguientes opciones:

-  `Thunderbolt BIOS Assist Mode`: **Disabled**. NO lo pongas en ninguna otra cosa, ya que se sabe que puede causar bricks potenciales en el X280, ¡que **no podrás solucionar en casa!**
-  `Wake by Thunderbolt(TM) 3`: **No**
-  `Security Level`: **No**
-  `Support in Pre Boot Environment > Thunderbolt(TM) device`: **No**

Ahora puedes proceder con la instalación.

### Instalar macOS
1. Arranca desde el USB, presiona ```SPACE``` y selecciona la unidad USB dentro de OpenCore ```"NO NAME (DMG)" o similar```.
>  **Nota:** El primer arranque puede tardar hasta 20 minutos.
2. Espera a la pantalla de Utilidades de macOS.
3. Conéctate a tu red si usas Wi-Fi.
4. Selecciona Utilidad de Discos, selecciona tu disco y haz clic en borrar. Dale un nombre y elige **APFS** con **Mapa de particiones GUID**.
5. Después de borrar, regresa y selecciona **Reinstalar macOS** y sigue los pasos en pantalla. La instalación puede tardar hasta **2 horas**; en mi caso, con una conexión de descarga de 300Mbps, tardó alrededor de una hora.
>  **Nota:** Tu PC se reiniciará varias veces. Si esto sucede (y pasará), simplemente arranca desde el USB nuevamente y selecciona tu disco dentro de OpenCore. Se llamará macOS Installer o el nombre que elegiste al formatear la unidad.
6. Una vez que veas la pantalla de `Selección de región`, puedes continuar.
7. Crea tu cuenta de usuario y todo lo demás. Deberías poder iniciar sesión con iCloud si generaste tu SMBIOS correctamente y cambiaste la dirección MAC.

</details>

<details>  
<summary><strong>♻️ Actualizar macOS / Cambiar EFI</strong></summary>
</br>

Si deseas actualizar macOS, descarga la versión de macOS deseada en la app de Ajustes y simplemente realiza la actualización como en un Mac real; sin embargo, si planeas actualizar tu EFI, necesitarás una configuración de OpenCore diferente.

> Nota: Descarga la versión de macOS deseada en Ajustes antes de seguir estos pasos, si estás conectado vía WiFi.

1. Descarga la versión más reciente y [ProperTree](https://github.com/corpnewt/ProperTree) y extráelo.
2. Inicia ProperTree y carga el ```Config.plist``` en tu partición EFI. (File -> Open)
> Nota: Puedes montar tu partición EFI presionando ```ALT + SPACE```, escribiendo Terminal e ingresando el siguiente comando: ```sudo diskutil mountDisk disk0s1```.
3. Ahora carga también el nuevo archivo de configuración del repositorio para la instalación de macOS deseada (o configuración de HeliPort). 
4. Ahora deberías tener 2 ventanas de ProperTree abiertas en tu pantalla.
5. Ve en ambas ventanas a ```Root -> PlatformInfo -> Generic```. Transfiere ```MLB, ROM, SystemProductName, SystemSerialNumber y SystemUUID``` a la nueva config. 
6. Guarda la nueva config (File -> Save) y cierra ambas ventanas.
7. Ahora borra tu carpeta EFI existente de la partición EFI y copia la nueva en ella. (Asegúrate de que los directorios ```Boot y OC``` estén dentro de ```EFI```).

</details>

&nbsp;

## Post-instalación (opcional)

<details>  
<summary><strong>💾 Instalar OpenCore en el Disco Duro (recomendado)</strong></summary>
</br>

1. Presiona `ALT + SPACE` y abre la terminal. Escribe `sudo diskutil mountDisk disk0s1` (donde disk0s1 corresponde a la partición EFI del disco principal; los discos se pueden listar usando el comando `sudo diskutil list`, pero lo más probable es que sea `disk0s1`)
2. Abre Finder y copia la carpeta EFI de tu dispositivo USB a la partición EFI del disco principal.
3. Desconecta el dispositivo USB y reinicia tu laptop. Ahora puedes arrancar macOS sin tu dispositivo USB.

</details>

<details>  
<summary><strong>✏️ Crear un medio de instalación offline (Opcional)</strong></summary>
</br>

En caso de reinstalar macOS, es útil tener un instalador completo del sistema en tu unidad USB. De esta manera, tampoco necesitarás conexión a internet para la instalación. Para crear un medio de instalación offline, necesitas lo siguiente: 

- Instalador de macOS desde la App Store.
- Un pendrive de 16 GB o más (Ten en cuenta que, durante la preparación, formatearemos el disco para crear el medio de instalación).

Descarga el instalador de la App Store. Presiona `ALT + SPACE` y abre la Utilidad de Discos. Selecciona tu dispositivo USB y haz clic en borrar. Nómbralo `MyUSB` y elige **Mac OS Extended** con **Mapa de particiones GUID**. Después de borrar el dispositivo USB, cierra la Utilidad de Discos.

Ahora presiona `ALT + SPACE` y abre la terminal. Escribe el siguiente comando:

Big Sur:
```sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/MyUSB --downloadassets```

Monterey:
```sudo /Applications/Install\ macOS\ Monterey.app/Contents/Resources/createinstallmedia --volume /Volumes/MyUSB --downloadassets```

Después de crear el medio de instalación, copia tu carpeta EFI a la partición EFI de tu dispositivo USB.


</details>

&nbsp;

## Estado

<details>  
<summary><strong>✅ Qué funciona</strong></summary>
</br>
 
- [X] WiFi y Bluetooth de Intel (gracias a [itlwn](https://github.com/OpenIntelWireless/itlwm))
- [X] Control de Brillo / Volumen
- [X] Información de Batería
- [X] Audio (Jack de Audio y Altavoz)
- [X] Puertos USB
- [X] Aceleración Gráfica
- [X] Trackpoint / Touchpad
- [X] Gestión de energía
- [X] Suspensión (puede ser inestable pero mayormente funciona)
- [X] FaceTime / iMessage (iServices)
- [X] HDMI
- [X] Actualizaciones automáticas del SO
- [X] Handoff / Portapapeles Universal
- [X] Sidecar (Cable) / AirPlay a Mac
- [X] SIP / FireVault 2 (aunque no recomendaría el cifrado debido a posibles problemas de soporte futuros)
- [X] USB-C

</details>

<details>  
<summary><strong>⚠️ Qué no funciona</strong></summary>
</br>

- [ ] DRM de Safari ```Usa un navegador basado en Chromium o Firefox para ver Amazon Prime Video, Netflix, Disney+ y otros```
- [ ] AirDrop y Continuidad
- [ ] Lector de huellas (Deshabilitado con kext NoTouchID + no puedo probarlo porque mi modelo no tiene uno)
- [ ] Thunderbolt 3
- [ ] Sidecar Inalámbrico
- [ ] Desbloqueo con Apple Watch

</details>

<details>  
<summary><strong>🔄 No probado</strong></summary>
</br>

- [ ] WWAN
- [ ] Dualboot Windows / Linux (con OpenCore)
- [ ] Cámara integrada

</details>

&nbsp;

## ⭐️ Comentarios
Si encuentras algún error o simplemente tienes algunas preguntas, no dudes en enviar tus comentarios utilizando la pestaña de Discussions.

&nbsp;

## 📜 Licencia

Este repositorio está bajo la [Licencia MIT](https://github.com/valnoxy/t480-oc/blob/main/LICENSE).

OpenCore está bajo la [Licencia BSD 3-Clause](https://github.com/acidanthera/OpenCorePkg/blob/master/LICENSE.txt).

<hr>
<h6 align="center">2023 - 20xx 0x8008. 
<br>
<h6 align="center">Basado mayormente en el trabajo realizado por <a href="https://github.com/valnoxy/t480-oc">valnoxy</a>. Gracias amigo, te debo una si nuestros caminos se cruzan en algún momento.
<br>
