# Debloat Fire TV Stick 3e gen — à lancer depuis un ordinateur sur le même Wi-Fi

Applis gardées : YouTube, Prime Video, mise en miroir. Recherche vocale gardée, extras Alexa coupés.
Uniquement `pm disable-user --user 0` (via tools/disable.sh), rien n'est désinstallé.

## 0. Préparation
```bash
git clone https://github.com/Gathin23/cleanyourtv && cd cleanyourtv
cp -r <chemin>/firetv-stick-3 lists/
python3 -m venv tools/venv && tools/venv/bin/pip install -r tools/requirements.txt
export TV_HOST=192.168.1.192
PY=tools/venv/bin/python
```

## 1. Probe (accepte « Autoriser le débogage USB ? » sur la télé)
```bash
$PY -I tools/tv.py --probe
```

## 2. Sauvegarde complète, avant toute modification
```bash
mkdir -p backup
$PY -I tools/tv.py 'pm list packages'          > backup/packages.txt
$PY -I tools/tv.py 'pm list packages -d'       > backup/disabled-avant.txt
$PY -I tools/tv.py 'pm list packages -3'       > backup/apps-tierces.txt
$PY -I tools/tv.py 'getprop'                   > backup/getprop.txt
$PY -I tools/tv.py 'settings list global'      > backup/settings-global.txt
$PY -I tools/tv.py 'settings list secure'      > backup/settings-secure.txt
$PY -I tools/tv.py 'settings list system'      > backup/settings-system.txt
```
Compare `backup/packages.txt` aux lots : retire les paquets absents, signale ceux qui sont inconnus.

## 3. Un lot à la fois, avec une vérification après chaque lot
```bash
tools/disable.sh lists/firetv-stick-3/lot1-telemetrie.txt
```
Après chaque lot, vérifie que l'accueil s'affiche, que Home, Retour et le D-pad répondent, que le Wi-Fi est
connecté, et que YouTube et Prime Video lisent une vidéo (`$PY -I tools/tv.py --screenshot shots/lotN.png`).
Au moindre souci, restaure tout de suite le lot et arrête-toi :
```bash
tools/restore.sh lists/firetv-stick-3/lotN-xxx.txt com.amazon.tv.launcher
```
Ordre : lot1 → lot2 → lot3 → lot4 → lot5 (`HANDLERS_ALLOW=1 tools/disable.sh ...`) → lot6 → lot7.
Ignore « Cannot disable a protected package » : Fire OS protège ce paquet, il reste actif.

## 4. Après le dernier lot
```bash
$PY -I tools/tv.py 'sync'; sleep 15; $PY -I tools/tv.py 'reboot'
# une fois la clé redémarrée :
$PY -I tools/tv.py 'pm list packages -d' > backup/disabled-apres.txt
```

## Restauration complète (commande exacte)
```bash
TV_HOST=192.168.1.192 tools/restore.sh lists/firetv-stick-3/tout.txt com.amazon.tv.launcher
```

## Pas des paquets, mais à faire (Paramètres → Préférences)
- Confidentialité : coupe les données d'utilisation de l'appareil, la collecte d'utilisation des applis et les pubs ciblées.
- Contenu en vedette : coupe la lecture auto de la vidéo et du son.
- Le grand bandeau défilant et les bandes Prime Video viennent du launcher : aucun paquet ne les supprime.

## Fin : coupe le débogage ADB
Paramètres → Ma Fire TV → Options pour les développeurs → Débogage ADB → OFF, puis :
```bash
unset TV_HOST; $PY -I tools/adb-closed.py 192.168.1.192   # doit répondre « refused »
```
