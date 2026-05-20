---
title: Problemas de filtrado en Chrome 142+ al usar el controlador TDI
sidebar_position: 1
---

:::info

Este artículo describe AdGuard para Windows v8.0, un bloqueador de anuncios integral que protege su dispositivo a nivel del sistema. Esta es una versión beta que aún está en desarrollo. Para probarlo, descarga la [versión beta de AdGuard para Windows](https://agrd.io/windows_beta).

:::

Algunos usuarios de AdGuard para Windows pueden notar que [la aplicación deja de filtrar el tráfico en navegadores basados en Chromium](https://github.com/AdguardTeam/AdguardForWindows/issues/5771). A partir de Google Chrome 142+, el tráfico del navegador simplemente no aparece a nivel del controlador TDI, lo que impide que AdGuard lo inspeccione o lo filtre.

Este comportamiento no es un error en AdGuard, sino el resultado de cambios recientes en la arquitectura y la seguridad de los navegadores modernos.

## Por qué sucede esto

Navegadores basados en Chromium (Chrome, Edge, Brave, Vivaldi, etc.) han estado fortaleciendo su arquitectura de seguridad. Un cambio significativo es trasladar los procesos internos sensibles al [sandbox de Windows AppContainer](https://learn.microsoft.com/en-us/windows/win32/secauthz/appcontainer-isolation), incluido el Network Service, que gestiona todo el tráfico del navegador.

### Qué cambió en Chrome 142

A partir de Chrome 142, el proceso Network Service ahora se inicia dentro de AppContainer de forma predeterminada.

Cuando esto ocurre, las aplicaciones que se ejecutan en un AppContainer no utilizan la interfaz de red TDI heredada; en su lugar, su tráfico se enruta a través de la pila más moderna WSK (Winsock Kernel). Como resultado, el controlador TDI no puede ver, interceptar ni procesar las conexiones que pasan por WSK, y todo el tráfico del navegador se vuelve invisible para el controlador TDI utilizado por AdGuard.

Este comportamiento está controlado completamente por las políticas de aislamiento (sandboxing) de Chrome y los experimentos internos (pruebas de campo), no por la configuración del usuario.

## Por qué esto afecta a AdGuard

El controlador TDI es una tecnología de Windows obsoleta que ha sido descontinuada y no es compatible por Microsoft desde hace muchos años. No es compatible con los modelos modernos de aislamiento y sandboxing utilizados por los navegadores.

Debido a esto, la visibilidad del tráfico basada en TDI se vuelve cada vez más inestable. En algunos navegadores, ya ha desaparecido por completo, y eventualmente dejará de funcionar por completo.

AdGuard ya considera obsoleto el controlador TDI, y su eliminación completa está prevista a medida que el producto evoluciona.

## Solución permanente

Desde la versión 8.0 RC, hemos añadido soporte experimental para el controlador SockFilter. Soluciona el problema al resolver conflictos en la stack WFP. [Más información](/adguard-for-windows-8/settings/app-settings/network-settings/).

Para usarlo, ve a _Configuración → Red → Filtrado de tráfico_, habilita el filtrado de tráfico y selecciona _SockFilter (Experimental)_ de la lista de opciones disponibles.

Como es experimental, puede haber errores. Si notas algo inusual, inesperado o simplemente roto, **puedes volver a TDI o WFP en cualquier momento** en la misma sección.

## Solución temporal

Ciertos cambios en el registro de Windows pueden obligar al navegador a dejar de usar AppContainer, provocando que sus procesos vuelvan a ejecutarse en un modo sin sandbox. El Servicio de red deja de usar la pila WSK y recurre a una ruta de red que el controlador TDI puede ver. AdGuard vuelve a tener la capacidad de filtrar el tráfico del navegador.

### Cómo modificar el registro en navegadores basados en Chromium

:::warning

Se requieren derechos de administrador para editar el registro. Los cambios incorrectos pueden afectar la estabilidad y la seguridad del sistema o del navegador. Siempre cree una copia de seguridad de la rama del registro antes de modificarla.

Antes de continuar, ten en cuenta que esta solución reduce la seguridad del sandbox/AppContainer, haciendo que el navegador esté menos aislado. Se aplica a todo el sistema porque modifica `HKLM`, y solo debe usarse para depuración, soluciones temporales, en entornos controlados o cuando la interceptación de tráfico basada en TDI sea estrictamente necesaria.

**No** debería aplicarse de forma generalizada en las máquinas de usuarios finales. **Continúa solo si entiendes las implicaciones.**

:::

You can apply the necessary registry changes automatically by using one of the pre-generated .reg files below. Each file disables AppContainer/Network Service sandboxing for a specific Chromium-based browser:

- [Download Chrome.reg](https://cdn.adtidy.org/distr/windows/reg/DisableAppContainer_Chrome.reg)
- [Download Chromium.reg](https://cdn.adtidy.org/distr/windows/reg/DisableAppContainer_Chromium.reg)
- [Download Edge.reg](https://cdn.adtidy.org/distr/windows/reg/DisableAppContainer_Edge.reg)
- [Download Brave.reg](https://cdn.adtidy.org/distr/windows/reg/DisableAppContainer_Brave.reg)
- [Download Vivaldi.reg](https://cdn.adtidy.org/distr/windows/reg/DisableAppContainer_Vivaldi.reg)
- [Download YandexBrowser.reg](https://cdn.adtidy.org/distr/windows/reg/DisableAppContainer_YandexBrowser.reg)

If your browser is not listed, follow the manual instructions below to create the necessary registry entries:

1. Determine its policy branch by checking the vendor’s official documentation or by opening the internal policy page. In Chrome, navigate to `chrome://policy`. Other browsers use a similar path.

2. Identify the correct registry branch for your browser. Different Chromium-based browsers use different policy paths under `HKLM`. It should follow the model `HKLM\SOFTWARE\Policies\<Vendor>\<Product>`.

3. Open the Registry Editor:

   - Press _Win + R_
   - Type _regedit_ and press _Enter_
   - Approve the UAC prompt by running it as administrator

4. Back up the Policies branch:

   - In the left panel, navigate to `HKEY_LOCAL_MACHINE\SOFTWARE\Policies`
   - Right-click _Policies_ → _Export_
   - Save the file as _Policies_backup.reg_

   If something goes wrong, you can restore the backup by double-clicking this file.

5. Navigate to your browser’s policy key:

   - Expand the path `HKEY_LOCAL_MACHINE` → _SOFTWARE_ → _Policies_.
   - Locate the folder corresponding to your browser.

If the key does not exist, you can create it manually. Example for Chrome:

- Right-click _Policies_ → _New_ → _Key_ and name it `Google`
- Inside `Google`, create another key named `Chrome`

Repeat the same logic for Chromium, Edge, Brave, Vivaldi, Yandex Browser, etc. You should end up with a key that looks like `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\<Vendor>\<Product>`.

1. Add the required registry values:

   - In the correct key, click the right panel → _New_ → _DWORD (32-bit) Value_

   - Name it `RendererAppContainerEnabled`

   - Double-click it and set:

     - **Value:** 0
     - **Base:** Hexadecimal or Decimal (either is fine)

   - Repeat the process and create a second DWORD `NetworkServiceSandboxEnabled`.

   - Ponle su valor a 0.

   Ambos parámetros deben ser 'REG_DWORD' y tener el valor **0**.

2. Cierra el navegador y aplica la configuración. Para asegurar que la política esté cargada:

   - Cierra completamente el navegador
   - Revisa el Administrador de tareas y asegúrate de que ningún proceso como _chrome.exe_, _msedge.exe_, _brave.exe_ siga ejecutándose
   - Vuelve a abrir el navegador

3. Verifica que las políticas se hayan aplicado abriendo el visor de políticas en tu navegador.

Deberías ver activas las siguientes políticas:

- 'RendererAppContainerEnabled' — **0 / false**
- 'NetworkServiceSandboxEnabled' — **0 / false**

Si está disponible, haz clic en _Recargar políticas_.

¡Listo!
