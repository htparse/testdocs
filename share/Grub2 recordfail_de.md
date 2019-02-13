##GRUB2 recordfail Feature deaktivieren
GRUB2 bringt ein Feature mit, daß bei einem fehlgeschlagenen Bootversuch automatisch beim nächsten Boot im Bootmenu anhält.

Dies kann unter Umständen unerwünscht sein. Je nach GRUB Version muß dazu entweder die Datei */etc/grub.d/00_header* geändert oder die Variable GRUB_RECORDFAIL_TIMEOUT in */etc/default/grub* gesetzt werden.

Suchen Sie zunächst in */etc/grub.d/00_header* nach

```
if [ ${recordfail} = 1 ]; then
   set timeout=-1
else
  set timeout=${GRUB_TIMEOUT}
fi
```

Falls diese exakt so vorhanden ist, versehen Sie diese mit Kommentarzeichen

```
#if [ \${recordfail} = 1 ]; then
#    set timeout=-1
#else
    set timeout=${GRUB_TIMEOUT}
#fi
```

Falls Sie stattdessen folgendes vorfinden:

```
if [ "\${recordfail}" = 1 ]; then
 set timeout=${GRUB_RECORDFAIL_TIMEOUT:--1}
else
 set timeout=${2}
fi
```

Editieren Sie die Datei /etc/default/grub und fügen dort die Zeile

`GRUB_RECORDFAIL_TIMEOUT=5`

für einen 5 Sekunden Timeout hinzu.

Zur Aktualisierung der Konfiguration führen Sie dann in jedem Fall ein

`update-grub`

aus. Nun wird GRUB2 immer den eingestellten Timeout benutzen.

###Manualles Zurücksetzen des recordfail Wertes
Der Wert für recordfail in grubenv kann manuell zurückgesetzt

`grub-editenv set recordfail=0`

oder entfernt werden

`grub-editenv unset recordfail`
