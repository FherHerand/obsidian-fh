---
tags:
  - sterling-connect-direct
  - assa
  - inventory
  - cdfa
  - file-agent
aliases:
---

# Ambiente

Ficha de referencia técnica con las rutas, nodos y parámetros específicos asignados a esta infraestructura.

> [!link] Operaciones y Procedimientos
> Para la ejecución de comandos, reinicios y administración, consultar: [Comandos - Sterling Connect Direct](obsidian://open?vault=obsidian-fh&file=IBM%20Products%2FSterling%2FConnect%20Direct%2FSterling%20Connect%20Direct%20-%20Comandos).

---

## 1. Servidores y Nodos

| Nodo            | IP / Host    | Rol    | Directorio Work                   | Archivo Initparm                                 |
| :-------------- | :----------- | :----- | :-------------------------------- | :----------------------------------------------- |
| `PAIDCPORAFS01` | `10.1.1.184` | Nodo 1 | `/u02/cdunix/work/PAIDCPORAFS01/` | `/u02/cdunix/ndm/cfg/PAIDCPORAFS01/initparm.cfg` |
| `PAIDCPORAFS02` | `10.1.1.185` | Nodo 2 | `/u02/cdunix/work/PAIDCPORAFS02/` | `/u02/cdunix/ndm/cfg/PAIDCPORAFS02/initparm.cfg` |

---

## 2. Directorios y Rutas Base

- **Directorio base de instalación (`CD_HOME`):** `/u02/cdunix/`
- **Binarios de Connect:Direct:** `/u02/cdunix/ndm/bin/`
- **Configuración CLI API (`ndmapi.cfg`):** `/u02/cdunix/ndm/cfg/cliapi/ndmapi.cfg`
- **Directorio base File Agent (`CDFA`):** `/u02/cdunix/file_agent/`

### Archivos Netmap
- **Nodo 1:** `/u02/cdunix/ndm/cfg/PAIDCPORAFS01/netmap.cfg`
- **Nodo 2:** `/u02/cdunix/ndm/cfg/PAIDCPORAFS02/netmap.cfg`

---

## 3. Ubicación de Logs y Estadísticas

### Connect:Direct
- **Logs de trabajo Nodo 1:** `/u02/cdunix/work/PAIDCPORAFS01/`
- **Logs de trabajo Nodo 2:** `/u02/cdunix/work/PAIDCPORAFS02/`

### File Agent
- **Log operativo:** `/u02/cdunix/file_agent/CDFA.log`
- **Log de estadísticas:** `/u02/cdunix/file_agent/CDFA_stats.log`