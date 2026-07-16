<div align="center">

# ◢ H O R I Z O N . C 2 ◣

[![Rust](https://img.shields.io/badge/Rust-000000?style=for-the-badge&logo=rust&logoColor=white)](https://www.rust-lang.org/)
[![Go](https://img.shields.io/badge/Go-00ADD8?style=for-the-badge&logo=go&logoColor=black)](https://go.dev/)
[![Windows](https://img.shields.io/badge/Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)](https://www.microsoft.com/windows)
![Red Team](https://img.shields.io/badge/Red_Team-red?style=for-the-badge)

**Command & Control Framework**
___
</div>

## Resumen

Horizon-C2 es un framework experimental de Comando y Control (C2) diseñado para operaciones de Red Team que demandan un control preciso de la huella en memoria.


## Ingeniería de Comunicación

- **Protocolo Binario Minimalista**: Eliminación de serializaciones de alto nivel como JSON o Protobuf. La transmisión de comandos opera bajo un protocolo binario directo, reduciendo drásticamente la firma estática en tráfico de red.

- **Transporte TLS Evasivo**: Implementación sobre `rustls` con inhabilitación del almacén de certificados del sistema. La integridad se garantiza mediante SSL Pinning con certificados ofuscados en tiempo de compilación, invalidando técnicas estándar de inspección MITM.

- **Validación Criptográfica Estricta**: Implementación de wrappers personalizados que permiten la operación mediante IP cruda, omitiendo los campos SAN (Subject Alternative Names) sin sacrificar la validación de la cadena de confianza.

## Interfaz del Operador

El Team-Server emplea una arquitectura modular concurrente. Su diseño permite el despliegue dinámico de comandos y la gestión de múltiples agentes con una latencia mínima, priorizando la estabilidad del canal de mando en entornos de operaciones prolongadas.


## Capacidades Operativas (OPSEC)

- **Comandos Nativos**: Integración directa con APIs de Windows. El framework evita la invocación de intérpretes de comandos (`cmd.exe` / `powershell.exe`), minimizando la telemetría de procesos sospechosos.

- **Estabilidad en Memoria**: Agente desarrollado íntegramente en Rust. Su arquitectura síncrona, libre de runtimes complejos (Tokio), asegura una huella de memoria mínima y predecible bajo condiciones de latencia variable.

- **PPID Spoofing**: Implementación de suplantación de ID de Proceso Padre para evadir detecciones basadas en parentesco, optimizada para la resiliencia en sistemas Windows modernos.
> **Nota**: Durante el desarrollo, se documentó que Windows 11 mitiga agresivamente el PPID Spoofing a nivel de kernel para ciertos procesos (ej. cmd.exe), revirtiendo silenciosamente el padre a `explorer.exe`. El framework evita esto apuntando el ppid directamente a `explorer.exe`.

## Comandos internos del agente
```
whoami
env
pwd 
ls
cd
mv
cp
rm
mkdir
download
upload
netstat
ps
ppid
run
kill
exit
```


## Notas de Implementación

El proyecto requiere openssl para la generación de la PKI y la ofuscación de certificados. El proceso de hardening mediante el script incluido en `tools/` es obligatorio para asegurar la integridad de la CA en el agente.

![](img/screenshot_2026-07-16-023052.png)
![](img/screenshot_2026-07-16-023142.png)
![](img/screenshot_2026-07-16-023230.png)
![](img/screenshot_2026-07-16-023356.png)
![](img/screenshot_2026-07-16-023525.png)
![](img/screenshot_2026-07-16-023714.png)


## Uso

Linux
```
sudo apt install llvm clang
cargo install cargo-xwin
rustup target add x86_64-pc-windows-msvc

```

1. Preparación de Certificados

El Agente utiliza SSL Pinning, por lo que necesita conocer de antemano el certificado del Team-Server.
```bash
# Generar par de llaves TLS para el C2
openssl req -x509 -newkey rsa:4096 -keyout team-server/cert/private.key -out team-server/cert/certificate.crt -days 365 -nodes
```
2. Ofuscación del Certificado

El Agente no confía en el certificado en texto claro, necesita que esté ofuscado en el código fuente. Ejecutar el script Python incluido en `tools/` pasándole el certificado generado anteriormente:
```bash
python3 tools/encodex_DER.py team-server/cert/certificate.crt
```
Esto imprimirá en la terminal el arreglo de bytes `ENCRYPTED_CA` y la llave `CA_XOR_KEY`. Debes copiarlas y  pegarlas directamente en el archivo `beacon/src/core/profile.rs`, reemplazando las que están por defecto.

3. Compilar el Team Server

Ingresar al directorio del servidor y compilar el binario de Go
```bash
cd team-server
go build main.go -o horizon-teamserver
```

4. Compilar el Beacon

Compilar el agente en Rust para Windows x64. (Asegúrate de tener el target `x86_64-pc-windows-gnu` instalado en tu `Rustup`)
```bash
cd beacon
cargo xwin build --release --target x86_64-pc-windows-msvc
```
El binario final ofuscado estará en: `beacon/target/x86_64-pc-windows-msvc/release/horizon-beacon.exe`.

5. Ejecución

Iniciar el Team Server en tu máquina atacante:
```bash
sudo ./horizon-teamserver
```

En el equipo víctima (Windows), ejecutar el beacon pasando la dirección del servidor como argumento:
```powershell
.\horizon-beacon.exe
```


## ⚠️ Descargo de Responsabilidad

Esta herramienta ha sido desarrollada para fines de investigación en seguridad.

El autor no se hace responsable del uso indebido de esta herramienta ni de las consecuencias derivadas de su utilización. 

> **Nota**: El proyecto puede incluir comentarios en su código fuente que podrían influir en la detección del binario resultante por parte de determinados sistemas de seguridad.

<div align="center">
Aukan  |  CRTO Certified
</div>


