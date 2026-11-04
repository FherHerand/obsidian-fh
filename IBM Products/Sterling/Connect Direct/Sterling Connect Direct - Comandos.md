---
tags:
  - middleware
  - sterling-connect-direct
  - file-agent
  - runbook
  - cheatsheet
  - cdfa
---

# Comandos y Operaciones

## 1. Verificaciones del Sistema Operativo

### Ulimits
Validar los límites configurados para el usuario de Connect:Direct:
```bash
head -2 /etc/security/limits.conf
```

---

## 2. Gestión del Servicio Connect:Direct

### 2.1. Verificación de Procesos
```bash
# Todos los procesos asociados a Connect:Direct
ps -ef | grep cdunix

# Proceso demonio initp
ps -ef | grep initp
```

### 2.2. Detención del Servicio
> [!tip] Buenas Prácticas
> Ejecutar siempre la detención controlada (*graceful*) antes de forzar señales `kill`.

**Método 1: Script de parada (Recomendado)**
```bash
cd /u02/cdunix/ndm/bin
sudo ./cduStop
```

**Método 2: Parada vía CLI direct**
```bash
cd /u02/cdunix/ndm/bin/
direct
> stop
```

**Método 3: Detención forzada (Fallback)**
```bash
ps -ef | grep initp
kill -9 <pids>
ps -ef | grep initp
```

### 2.3. Inicio del Servicio

**Método 1: Por archivo de configuración de nodo**
```bash
# Iniciar Nodo 1
/u02/cdunix/ndm/bin/cdpmgr -i /u02/cdunix/ndm/cfg/PAIDCPORAFS01/initparm.cfg

# Iniciar Nodo 2
/u02/cdunix/ndm/bin/cdpmgr -i /u02/cdunix/ndm/cfg/PAIDCPORAFS02/initparm.cfg
```

**Método 2: Script de inicio estándar**
```bash
cd /u02/cdunix/ndm/bin
./cduStart
```

### 2.4. Mantenimiento y Rotación del Directorio Work
En caso de corrupción o necesidad de reinicio limpio del spool de trabajo:
```bash
cd /u02/cdunix/work

# Nodo 1
mv PAIDCPORAFS01 PAIDCPORAFS01_BKP && mkdir -m 1775 PAIDCPORAFS01

# Nodo 2
mv PAIDCPORAFS02 PAIDCPORAFS02_BKP && mkdir -m 1775 PAIDCPORAFS02
```

---

## 3. Logs y Monitoreo

### Monitoreo en Tiempo Real del Último Log Generado
```bash
# Nodo 1
LAST_FLOG=$(ls -t /u02/cdunix/work/PAIDCPORAFS01/S* | grep -v ".gz" | head -n 1)
echo "Monitoring: $LAST_FLOG"
tail -f "$LAST_FLOG"

cat $LAST_LOG | grep "copy step succ" -i


# Nodo 2
LAST_FLOG=$(ls -t /u02/cdunix/work/PAIDCPORAFS02/S* | grep -v ".gz" | head -n 1)
echo "Monitoring: $LAST_FLOG"
tail -f "$LAST_FLOG"
```

### Versión del Producto
```bash
cd /u02/cdunix/etc/
./cdver
```

---

## 4. Consola CLI (`direct`)

Exportar la variable de entorno obligatoria antes de ingresar a la consola:
```bash
NDMAPICFG=/u02/cdunix/ndm/cfg/cliapi/ndmapi.cfg
export NDMAPICFG
direct
```

### Operaciones Comunes en CLI:
```text
/* Limpieza y cancelación de procesos en cola */
FLUSH PROCESS;
flush process pname=PDFCOPY;
flush process pname=PDFCOPY force=yes;

/* Salir de la CLI */
quit;
```

---

## 5. Configuración Secure+ (`spcli.sh`)

Acceso a la herramienta interactiva de Secure+:
```bash
cd /u02/cdunix/ndm/bin/
./spcli.sh
```

### Comandos frecuentes en `spcli`:
```text
/* Modificar protocolo del cliente */
update client protocol=disable;
/* update client override=y; */

/* Consultar configuración */
display remotenode name=.Client;

quit;
```

---

## 6. File Agent (`CDFA`)

### 6.1. Verificación de Estado
```bash
ps -ef | grep cdfa
```

### 6.2. Detención
```bash
# Parada controlada
cd /u02/cdunix/file_agent
./cdfastop

# Fallback forzado
ps -ef | grep cdfa
kill -9 <pid>
```

### 6.3. Inicio y Consola
```bash
# Iniciar en segundo plano
cd /u02/cdunix/file_agent/
./cdfa > /dev/null 2>&1 &

# Consultar versión
cd /u02/cdunix/file_agent/
./cdfa -p

# Abrir en modo consola
cd /u02/cdunix/file_agent/
./cdfa -C
```

### 6.4. Monitoreo de Trazas
```bash
tail -f /u02/cdunix/file_agent/CDFA.log
tail -f /u02/cdunix/file_agent/CDFA_stats.log
```

---

## 7. Referencias y Documentación Oficial

- [IBM Docs: Detención de Connect:Direct](https://www.ibm.com/docs/en/connect-direct/6.3.0?topic=guide-stopping-connectdirect)
- [IBM Docs: Uso de la CLI de Connect:Direct](https://www.ibm.com/docs/en/connect-direct/6.3.0?topic=guide-using-cli)
- [IBM Docs: Catálogo de Comandos C:D](https://www.ibm.com/docs/en/connect-direct/6.3.0?topic=guide-connectdirect-commands)
- [IBM Support: Errores XSSF021I, XTQN001I y XTQC0021 en logs de estadísticas](https://www.ibm.com/support/pages/error-messages-xrrf021i-xtqn001i-and-xtqc0021-seen-connectdirect-unix-statistics-log?view=full)
- [IBM Support: Fallo de proceso ejecutado en script shell (XCPS002I)](https://www.ibm.com/support/pages/process-submiited-within-shell-script-fails-xcps002i)