```
Techarisma Chapter 5/7

We have collected a memory dump of Jacob's compromised system. Can you find out how the attackers have planned to come back in case they lost access?

File: memdump.raw
```

Beskrivelsen av oppgaven hinter til "persistence" i systemet

Populære steder for å skjule seg er i:
```
-   HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run  
-   HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce 
-   HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run 
-   HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

Bruker volatility3 for å se på disse lokasjonene og i `\CurrentVersion\RunOnce` finner en encoded powershell skript
```
python3 vol.py -f memdump.raw windows.registry.printkey.PrintKey --key "Software\Microsoft\Windows\CurrentVersion\RunOnce"
```

Åpner en clean window vm og disabler "RealtimeMonitoring" og kjører skriptet 
`Set-MpPreference -DisableRealtimeMonitoring $false`

og får ut flagget 

**EPT{the_action_or_fact_of_persisting}**
