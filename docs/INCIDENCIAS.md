# Incidencias y soluciones

| # | Problema | Causa | Solución |
|---|----------|-------|----------|
| 1 | Al imprimir una cotización sale HTML y un aviso naranja "Unable to find Wkhtmltopdf" | Ubuntu 26.04 no trae `wkhtmltopdf` en sus repositorios | Instalar el `.deb` de jammy (0.12.6.1-3) desde github.com/wkhtmltopdf/packaging con `sudo apt install ./archivo.deb` |
| 2 | Tras instalar wkhtmltopdf seguía el aviso | Odoo detecta el binario una sola vez al arrancar | Reiniciar el servidor de Odoo |
| 3 | La base de datos debe ser UTF8 | Requisito de Odoo | `createdb -E UTF8 -T template0 --locale=C.UTF-8` |
| 4 | "Oh snap!": no se puede cambiar la moneda de la compañía | Ya existen apuntes contables | Elegir país/moneda al crear la BD (en Docker se creó con Perú → PEN) o no cambiarla después |
| 5 | RUC rechazado en la BD con localización Perú | El RUC se valida con dígito verificador | Usar un RUC válido (ej. 20123456786) |
| 6 | Al detener con Ctrl+C aparece `psycopg2.InterfaceError: connection already closed` | Hilos cron al cerrar con Python 3.14 | Inofensivo; los datos no se afectan. Alternativa: usar Python 3.12 |
| 7 | Impuesto "15 %" en la BD de WSL | Datos de demostración de la localización genérica | Es solo demo; la BD Docker usa IGV 18 % (localización Perú) |
| 8 | Docker Desktop no muestra puerto en la fila del proyecto | Los puertos pertenecen a cada contenedor | Ver el contenedor `odoo-1`: 8070:8069 |
