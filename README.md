# Parcial 3 – Despliegue de Odoo 18 en servidor local

Curso: Arquitectura de Software – UNAS

## Integrantes
- Salazar Ramos, Jhordy Antonio
- Alvarado Ichpas, Benny Joshua
- Yanac Osorio, Melor Smith
- Herrera Isidro, Franco Andre

## Qué contiene
Odoo 18.0 levantado de dos formas, ambas verificadas (app Ventas + cotización en PDF):

| Modo | Puerto | Base de datos |
|------|--------|---------------|
| **A. Docker Compose** (recomendado, puntos extra) | 8070 | PostgreSQL 16 en contenedor, BD `odoo_docker`, localización Perú |
| **B. Código fuente en WSL (Ubuntu)** | 8069 | PostgreSQL local, BD `odoo_unas` |

El código de Odoo **no** está en este repositorio: se usa la imagen oficial `odoo:18.0` (modo A) o se clona de `github.com/odoo/odoo`, rama 18.0 (modo B).

---

## A. Con Docker (3 comandos)

**Requisitos:** Docker Desktop instalado y abierto (con integración WSL activada si usas WSL) y Git.

### Paso 1. Verificar herramientas
```bash
git --version
docker --version
docker compose version
```
Los tres deben mostrar una versión. Si `docker` falla, abrir Docker Desktop y esperar a que diga "Engine running".

### Paso 2. Clonar el repositorio
```bash
git clone <URL-DEL-REPO> odoo-despliegue-unas
cd odoo-despliegue-unas
```

### Paso 3. (Opcional) Cambiar claves o puerto
```bash
cp .env.example .env
nano .env        # editar POSTGRES_PASSWORD o ODOO_PORT si se desea
```
Si se omite este paso se usan los valores por defecto (puerto 8070).

### Paso 4. Levantar los servicios
```bash
docker compose up -d
docker compose ps
```
La primera vez descarga las imágenes (unos minutos). `docker compose ps` debe mostrar `db-1` y `odoo-1` en estado *Up*.

### Paso 5. Crear la base de datos en el navegador
1. Abrir http://localhost:8070
2. Completar el formulario:
   - **Master Password:** una contraseña que recuerdes (sirve para administrar bases).
   - **Database Name:** `odoo_docker`
   - **Email / Password:** tu usuario de acceso (el correo puede ser inventado).
   - **Language:** Español (PE) o English.
   - **Country:** **Peru** (instala IGV 18 %, validación de RUC y moneda PEN).
   - Dejar sin marcar "Demo data" si quieres una base limpia.
3. Pulsar **Create database**. Tarda 1–3 minutos.

### Paso 6. Probar Ventas y el PDF
1. En Aplicaciones instalar **Ventas** (Sales).
2. Ventas → Cotizaciones → Nuevo. Elegir cliente (con RUC válido, p. ej. `20123456786`), agregar productos con impuesto **IGV 18%**.
3. Confirmar la cotización y usar **Imprimir** → debe descargarse un PDF.

### Comandos del día a día
| Acción | Comando |
|--------|---------|
| Ver estado | `docker compose ps` |
| Detener (conserva datos) | `docker compose stop` |
| Iniciar de nuevo | `docker compose start` |
| Ver registros de Odoo | `docker compose logs -f odoo` |
| Reiniciar solo Odoo | `docker compose restart odoo` |

> **Nunca usar `docker compose down -v`**: borra las bases de datos. `docker compose down` (sin `-v`) solo elimina los contenedores y conserva los datos.

---

## B. Desde código fuente en WSL (Ubuntu)

**Requisitos:** Windows con WSL2 y Ubuntu. Todos los comandos se ejecutan en la terminal de Ubuntu.

### Paso 1. Instalar PostgreSQL y dependencias
```bash
sudo apt update
sudo apt install -y postgresql git python3-venv python3-dev build-essential \
  libpq-dev libldap2-dev libsasl2-dev libxml2-dev libxslt1-dev libjpeg-dev wget
sudo systemctl enable --now postgresql
systemctl is-active postgresql      # debe responder: active
```
Si `systemctl` no funciona en tu WSL, usar `sudo service postgresql start`.

### Paso 2. Crear el rol y la base de datos (UTF8)
Cambiar `<CLAVE>` por una contraseña tuya:
```bash
sudo -u postgres psql -c "CREATE ROLE odoo WITH LOGIN CREATEDB PASSWORD '<CLAVE>';"
sudo -u postgres createdb -O odoo -E UTF8 -T template0 --locale=C.UTF-8 odoo_unas
psql -h 127.0.0.1 -U odoo -d odoo_unas -c "SHOW server_encoding;"   # debe mostrar UTF8
```

### Paso 3. Clonar Odoo 18 y crear el entorno virtual
```bash
mkdir -p ~/odoo18 && cd ~/odoo18
git clone --depth 1 -b 18.0 https://github.com/odoo/odoo.git
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip wheel setuptools
pip install -r odoo/requirements.txt
mkdir custom_addons
```

### Paso 4. Archivo de configuración
Copiar la plantilla desde este repositorio y editarla (reemplazar `<...>` con tus valores; `db_password` = la `<CLAVE>` del paso 2):
```bash
cp <RUTA-DEL-REPO>/odoo.conf.example ~/odoo18/odoo.conf
nano ~/odoo18/odoo.conf
```
Para ver tu usuario: `echo $USER`.

### Paso 5. Inicializar la base de datos
```bash
cd ~/odoo18
source venv/bin/activate
./odoo/odoo-bin -c odoo.conf -d odoo_unas -i base --stop-after-init
```
Termina solo tras unos minutos. Si no hay errores (`grep -c -E "ERROR|CRITICAL" odoo.log` → 0), continuar.

### Paso 6. Instalar wkhtmltopdf (para PDF)
Ubuntu 26.04 no lo trae en sus repositorios; se usa el paquete de Ubuntu 22.04:
```bash
cd /tmp
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.jammy_amd64.deb
sudo apt install -y ./wkhtmltox_0.12.6.1-3.jammy_amd64.deb
wkhtmltopdf --version
```

### Paso 7. Arrancar Odoo
```bash
cd ~/odoo18
source venv/bin/activate
./odoo/odoo-bin -c odoo.conf
```
Abrir http://localhost:8069 — usuario `admin`, contraseña `admin` (cambiarla después). Para detener: `Ctrl+C`.

### Paso 8. Probar Ventas y el PDF
Aplicaciones → instalar **Ventas** → crear cotización → confirmar → Imprimir → se descarga el PDF.
Si aparece un aviso "Unable to find Wkhtmltopdf", **reiniciar Odoo** (Odoo detecta el binario solo al arrancar).

---

## Estructura del repositorio
- `docker-compose.yml`, `.env.example` – despliegue con Docker
- `addons/` – carpeta montada en `/mnt/extra-addons` (módulos del Parcial 4)
- `odoo.conf.example` – plantilla de configuración para el modo B
- `docs/INCIDENCIAS.md` – problemas encontrados y soluciones
- `docs/capturas/` – evidencias

## Problemas frecuentes
Ver `docs/INCIDENCIAS.md`.

## Nota
Elaborado con apoyo de herramientas de IA, según lo autorizado por el docente.
