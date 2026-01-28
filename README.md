🛡️ Proyecto de Auditoría WiFi y Esteganografía
Este repositorio documenta el proceso técnico para la auditoría de seguridad de una red WPA2 y la post-explotación utilizando técnicas de esteganografía para ocultar información confidencial.

📋 Requisitos Previos
Sistema Operativo: Kali Linux

Herramientas: aircrack-ng, john (John the Ripper), steghide.

Archivos: Captura de tráfico (.cap) y una imagen de cobertura (.jpg).

🚀 FASE 1: Preparación del Entorno
Generamos el diccionario de fuerza bruta y preparamos el mensaje confidencial.

Bash
# 1. Crear diccionario de contraseñas (incluyendo la clave objetivo)
echo "12345678" > diccionario.txt

# 2. Crear el archivo con información sensible
echo "CONFIDENCIAL: Credenciales de acceso al servidor: admin/998877" > secreto.txt

# 3. Preparar la imagen de cobertura (asegúrate de tener una imagen .jpg)
mv imagen_original.jpg evidencia.jpg
🔓 FASE 2: Procesamiento y Cracking (WPA2)
Debido a limitaciones de hardware, se optó por limpiar el handshake y convertirlo a un formato compatible con John the Ripper.

Bash
# 1. Limpiar la captura .cap y extraer solo el Handshake válido
aircrack-ng -J handshake_limpio prueba_final-01.cap

# 2. Convertir el archivo .hccap a formato legible para John
hccap2john handshake_limpio.hccap > hash.txt

# 3. Ejecutar ataque de fuerza bruta con diccionario
john --wordlist=diccionario.txt hash.txt

# 4. Visualizar la contraseña obtenida
john --show hash.txt
Resultado esperado: proyecto:12345678

🕵️ FASE 3: Esteganografía (Post-Explotación)
Utilizamos la contraseña obtenida en la fase anterior (12345678) para ocultar el archivo secreto.txt dentro de evidencia.jpg.

1. Ocultamiento (Embedding)
Bash
# Incrustar el secreto en la imagen protegida por contraseña
steghide embed -cf evidencia.jpg -ef secreto.txt -p 12345678

# Eliminar el rastro (borrar el archivo de texto original)
rm secreto.txt
2. Extracción y Verificación
Para demostrar la integridad de los datos, extraemos el contenido oculto.

Bash
# Extraer la información oculta usando la clave recuperada
steghide extract -sf evidencia.jpg -p 12345678

# Leer el contenido del archivo recuperado
cat secreto.txt
⚠️ Disclaimer
Este proyecto fue realizado con fines puramente académicos y educativos en un entorno controlado y autorizado.
