# LANCast - Chat de voz LAN con música compartida

**Tipo:** App móvil multiplataforma (Android / iOS)  
**Stack:** Flutter (UI + lógica de control) + C++ (núcleo de audio y red)  
**Modo:** 100 % offline (LAN o hotspot)  
**Objetivo:** Chat de voz local con canal compartido de música donde cualquiera puede ser el DJ

---

## ARQUITECTURA GENERAL

### Flutter (Dart)
```

├─ UI/UX (Sliders, botones, chat)
├─ Gestor de usuarios y roles (host, DJ)
├─ Descubrimiento de peers (mDNS/UDP)
└─ Control de volúmenes y estados
🔻 FFI bridge (Flutter ↔ C++)

```

### Core nativo (C++ Engine)
```

├─ Captura de micrófono (PortAudio)
├─ Reproducción de audio (PortAudio)
├─ Mezclador de fuentes (voz + música)
├─ Codificador Opus
├─ Envío de paquetes (UDP multicast)
├─ Recepción y decodificación (Opus)
└─ Mixer local por usuario (volúmenes)

````

---

## MÓDULOS PRINCIPALES

### 1. Core de Audio (C++)
- Captura micrófono, reproduce audio, mezcla voz + música  
- Codifica/decodifica con Opus  
- Usa UDP multicast (239.255.x.x) para broadcast LAN  
**Librerías:** PortAudio, libopus, asio, libsndfile

### 2. Gestor de Red LAN
- Descubre peers vía mDNS o broadcast UDP  
- Mantiene tabla de peers activos  
- Establece conexión directa (por IP local)

### 3. Roles del canal
- **Speaker:** voz  
- **Listener:** solo escucha  
- **DJ:** transmite música local  

**Mensajes JSON:**
```json
{
  "type": "role_change",
  "from": "andres",
  "new_role": "dj"
}
````

### 4. Mixer local (C++)

* Mezcla canales independientes (voz/música)
* Control individual de volumen desde Flutter

**Ejemplo Dart:**

```dart
AudioEngine.setVolume(peerId: 'andres', volume: 0.6);
```

### 5. UI/UX (Flutter)

* Pantalla principal (usuarios y roles)
* Panel de volumen
* Botón DJ
* Estado de conexión LAN

**Paquetes:** udp_socket, multicast_dns, ffi, riverpod

### 6. Sincronización de música

* Opcional: sincronizar timestamps o transmitir directamente como canal de audio

---

## FLUJO DE OPERACIÓN

1. **Inicio**

   * App lanza broadcast UDP anunciando canal LANCast
   * Muestra peers conectados

2. **Conexión**

   * Todos escuchan canal multicast (239.255.42.69:5000)

3. **Transmisión**

   * Voz: envía paquetes Opus
   * Música: canal separado (239.255.42.70)

4. **Recepción**

   * Mezclador combina streams y ajusta volúmenes

5. **Cambio de DJ**

   * Nuevo DJ notifica con paquete control JSON

---

## FORMATO DE PAQUETES UDP

**Audio packet:**

```json
{
  "type": "audio",
  "peer_id": "andres",
  "stream": "voice",
  "seq": 1223,
  "timestamp": 169705920,
  "data": "<bytes opus>"
}
```

**Control packet:**

```json
{
  "type": "control",
  "action": "role_change",
  "peer_id": "carlos",
  "role": "dj"
}
```

---

## BLUEPRINT DEL PROYECTO

```
lancast/
├── lib/
│   ├── main.dart
│   ├── ui/
│   │   ├── home_screen.dart
│   │   ├── volume_panel.dart
│   │   └── peer_list.dart
│   ├── services/
│   │   ├── network_service.dart
│   │   ├── audio_service.dart
│   │   └── control_service.dart
│   ├── models/
│   │   ├── peer.dart
│   │   ├── roles.dart
│   │   └── message.dart
│   ├── providers/
│   │   └── app_state.dart
│   └── ffi/
│       ├── audio_bridge.dart
│       └── network_bridge.dart
│
├── native/
│   ├── CMakeLists.txt
│   ├── include/
│   │   ├── audio_engine.h
│   │   ├── network_manager.h
│   │   └── mixer.h
│   ├── src/
│   │   ├── audio_engine.cpp
│   │   ├── network_manager.cpp
│   │   └── mixer.cpp
│   └── third_party/
│       ├── portaudio/
│       ├── opus/
│       └── asio/
│
├── assets/
│   └── icons/
│
└── pubspec.yaml
```

---

## STACK RESUMIDO

**UI:** Flutter
**Lógica:** Dart
**Audio Engine:** C++ (PortAudio + Opus)
**Red LAN:** UDP multicast / mDNS
**Bridge:** FFI (Dart ↔ C++)
