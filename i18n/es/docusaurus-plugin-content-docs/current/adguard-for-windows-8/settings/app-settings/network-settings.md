---
title: Configuración de red
sidebar_position: 3
---

:::info

Este artículo describe AdGuard para Windows v8.0, un bloqueador de anuncios integral que protege su dispositivo a nivel del sistema. Esta es una versión beta que aún está en desarrollo. Para probarlo, descarga la [versión beta de AdGuard para Windows](https://agrd.io/windows_beta).

:::

This section lets you manage how AdGuard filters your Internet traffic and provides additional tools to control network-related features.

![Network settings](https://cdn.adtidy.org/content/kb/ad_blocker/windows/version_8/settings/network_settings.png)

## Use AdGuard as an HTTP proxy

Enable this option to make AdGuard work as an HTTP proxy. All traffic passing through it will be filtered — not only on your computer but also on other devices in your home network.

To filter traffic on other devices, install AdGuard’s root certificate on them and set up the proxy manually.

Other settings are enabled by default.

## Filtrado HTTPS

Most websites use HTTPS encryption today. Without HTTPS filtering, AdGuard cannot block ads on websites like YouTube, Facebook, or X (Twitter).

We recommend keeping the _Filter HTTPS_ option turned on unless you have a specific reason to turn it off.

With this option, you can:

- Reinstall or download the AdGuard certificate

- Add websites to a list that won’t be filtered

- Choose whether to filter websites with EV (Extended Validation) certificates

## Proxy for app and filter updates

Here you can choose which proxy AdGuard should use to download filter updates or new app versions — the system proxy, a custom proxy, or no proxy at all.

![Proxy settings](https://cdn.adtidy.org/content/kb/ad_blocker/windows/version_8/settings/proxy_apps.png)

## Use SockFilter driver

Enable this option to use the SockFilter driver, which helps block ads and trackers in apps on Windows 8 and later. If this option is disabled, AdGuard will switch to the older WFP driver instead.

SockFilter was first introduced in v8.0 RC. It is an _experimental_, lightweight kernel-mode network driver that works at the socket level (TCP/UDP). Instead of inspecting or modifying packets as they travel through the full Windows networking stack, a sock filter intercepts socket calls (e.g., connect, send, receive, bind) at a higher, more stable abstraction level. This makes it ideal for applications that need to monitor or control network activity without deep packet processing.

When fully tested and implemented, SockFilter has the potential to bring several advantages over other drivers:

- **It operates at a higher, socket-level layer**: SockFilter works with socket operations rather than raw packets, making it less complex and more stable than WFP’s low-level packet filtering.
- **No interference with other network drivers**: Because it sits above VPN, firewall, and antivirus WFP filters, it avoids filter-ordering problems and compatibility conflicts common in the WFP stack.
- **Greatly reduced risk of NETIO-related BSODs**: SockFilter doesn’t run inside the NETIO packet pipeline, so it avoids the typical crash scenarios caused by WFP callouts mishandling buffers, classification results, or packet memory.

En cuanto a desventajas, el controlador SockFilter solo detecta operaciones a nivel de socket y no captura tráfico generado por otros controladores del kernel o componentes que eluden la API estándar de Winsock. Desde una perspectiva de red de bajo nivel, esto puede considerarse una limitación, ya que el controlador no puede acceder a paquetes en bruto ni inspeccionar tráfico no basado en sockets. Sin embargo, para una aplicación de bloqueo de anuncios, este comportamiento no solo es aceptable sino óptimo. Todo el tráfico relevante de los navegadores y aplicaciones en modo usuario pasa por sockets estándar, y eso es exactamente lo que necesitamos controlar. Al mismo tiempo, ignorar el tráfico de controladores de bajo nivel elimina la complejidad innecesaria, evita problemas de compatibilidad y mantiene el sistema estable.
