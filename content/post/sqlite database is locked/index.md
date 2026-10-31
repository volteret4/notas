---
title : "SQLite database is locked con dos scripts a la vez"
slug : "sqlite-database-is-locked"
date : "2026-10-31"
image : ""
tags : ["python", "sqlite", "scripts"]
categories : ["scripts"]
description : "El indexador de dotfiles fallaba con database is locked porque el editor cambiaba el modo de journal de la base de datos."
---

# El problema

`~/Scripts/menus/dotfiles/` tiene dos programas que usan la misma base de datos `dotfiles.db`:

- `dotfiles_indexer.py`: indexa los archivos de `~/.local/share/chezmoi`.
- `dotfiles_quick_editor.py`: la interfaz PyQt6 que busca en la base de datos.

Si el editor estaba abierto, el indexador fallaba con:

```
sqlite3.OperationalError: database is locked
```

# La causa

El indexador abre la base de datos en modo **WAL**, que deja leer y escribir a la vez. El editor, al conectarse, la pasaba a `journal_mode = MEMORY` con `synchronous = OFF`. El modo de journal se guarda en el propio archivo, así que el editor quitaba el WAL y bloqueaba al indexador.

Además, ninguna conexión tenía `timeout`. Por defecto SQLite espera solo 5 segundos.

# Cambios

`dotfiles_indexer.py`, en `setup_database()`:

```diff
-        self.conn = sqlite3.connect(self.db_path)
+        self.conn = sqlite3.connect(self.db_path, timeout=30)
```

`dotfiles_quick_editor.py`, en `DatabaseSearchThread.run()` y en `setup_database()`:

```diff
-            conn = sqlite3.connect(str(self.db_path))
+            conn = sqlite3.connect(str(self.db_path), timeout=30)
...
-        self.conn = sqlite3.connect(str(self.db_path))
+        self.conn = sqlite3.connect(str(self.db_path), timeout=30)
...
-        # Optimizaciones para lectura
-        self.conn.execute('PRAGMA synchronous = OFF')
-        self.conn.execute('PRAGMA journal_mode = MEMORY')
+        # Optimizaciones para lectura — mantener WAL para compatibilidad con el indexer
+        self.conn.execute('PRAGMA synchronous = NORMAL')
+        self.conn.execute('PRAGMA journal_mode = WAL')
```

Regla general: si varios procesos usan la misma base de datos SQLite, **todos** deben usar `WAL` y un `timeout` amplio.

# Comprobar

```bash
sqlite3 ~/Scripts/menus/dotfiles/dotfiles.db 'PRAGMA journal_mode;'
```

Debe responder `wal`.

# Revertir

Los cambios no están en ningún commit de `~/Scripts`, así que `git diff` los muestra:

```bash
git -C ~/Scripts diff -- menus/dotfiles/
```

Para deshacerlos:

```bash
git -C ~/Scripts checkout -- menus/dotfiles/dotfiles_indexer.py menus/dotfiles/dotfiles_quick_editor.py
```
