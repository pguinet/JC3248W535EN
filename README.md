# JC3248W535EN - Démos ESP32-S3 avec écran LCD

Ce dépôt contient des démonstrations pour la carte **JC3248W535EN**, une carte de développement basée sur l'ESP32-S3 avec :

- Écran LCD 320x480 pixels (contrôleur AXS15231B, interface QSPI)
- Écran tactile capacitif
- Lecteur de carte SD/TF
- Sortie audio I2S
- 8 Mo de PSRAM

## Démos disponibles

| Démo | Description |
|------|-------------|
| **DEMO_LVGL** | Interface graphique avec la bibliothèque LVGL (widgets, benchmark, etc.) |
| **DEMO_PIC** | Visionneuse d'images JPEG (depuis carte SD) |
| **DEMO_MJPEG** | Lecteur vidéo MJPEG (depuis carte SD) |
| **DEMO_MP3** | Lecteur audio MP3 (depuis carte SD) |

## Installation des outils

### Option 1 : PlatformIO (recommandé)

PlatformIO est un environnement de développement embarqué qui s'installe facilement.

**Installation sur Linux :**
```bash
# Avec pipx (recommandé)
pipx install platformio

# Ou avec pip
pip install --user platformio
```

**Installation sur Windows/macOS :**
- Installer [VS Code](https://code.visualstudio.com/)
- Installer l'extension "PlatformIO IDE" depuis le marketplace

**Permissions série (Linux uniquement) :**
```bash
sudo usermod -a -G dialout $USER
# Redémarrer la session pour appliquer
```

### Option 2 : Arduino IDE

1. Télécharger [Arduino IDE](https://www.arduino.cc/en/software)
2. Ajouter le support ESP32 : Fichier → Préférences → URLs de gestionnaire de cartes :
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
3. Outils → Gestionnaire de cartes → Installer "esp32" (version 3.0.x)
4. Copier le contenu de `1-Demo/Demo_Arduino/libraries/` dans le dossier Arduino libraries

## Compiler et charger une démo

### Avec PlatformIO

1. **Connecter la carte** via USB

2. **Aller dans le dossier de la démo :**
   ```bash
   cd 1-Demo/Demo_Arduino/DEMO_LVGL
   ```

3. **Compiler :**
   ```bash
   pio run
   ```

4. **Téléverser sur la carte :**
   ```bash
   pio run -t upload
   ```

5. **Voir la sortie série (optionnel) :**
   ```bash
   pio device monitor
   ```

### Avec Arduino IDE

1. Ouvrir le fichier `.ino` de la démo (ex: `DEMO_LVGL/DEMO_LVGL.ino`)
2. Sélectionner la carte : Outils → Type de carte → ESP32S3 Dev Module
3. Configurer :
   - PSRAM: "OPI PSRAM"
   - Flash Mode: "QIO 80MHz"
   - USB CDC On Boot: "Enabled"
4. Cliquer sur Téléverser

## Changer de démo LVGL

Dans `DEMO_LVGL/src/main.cpp` (PlatformIO) ou `DEMO_LVGL/DEMO_LVGL.ino` (Arduino), modifier la fonction `setup()` :

```cpp
// Décommenter UNE seule ligne :
lv_demo_widgets();      // Interface avec onglets Profile/Analytics/Shop
// lv_demo_benchmark(); // Test de performance graphique
// lv_demo_music();     // Interface lecteur de musique
// lv_demo_stress();    // Test de stress
```

## Structure du dépôt

```
1-Demo/Demo_Arduino/
├── DEMO_LVGL/          # Démo interface graphique LVGL
├── DEMO_PIC/           # Démo visionneuse JPEG
├── DEMO_MJPEG/         # Démo lecteur vidéo
├── DEMO_MP3/           # Démo lecteur audio
├── libraries/          # Bibliothèques Arduino requises
└── TF file/            # Structure des dossiers pour carte SD

2-Specification/        # Spécifications hardware (PDF)
4-Driver_IC_Data_Sheet/ # Datasheets des composants
5-IO pin distribution/  # Schémas de brochage
6-User_Manual/          # Guide de démarrage
```

## Préparation de la carte SD (pour DEMO_PIC, DEMO_MJPEG, DEMO_MP3)

Créer les dossiers suivants à la racine de la carte SD :
- `/pic/` - Images JPEG
- `/mjpeg/` - Vidéos MJPEG
- `/music/` - Fichiers MP3

## Dépannage

**Le port série n'est pas détecté :**
- Linux : Vérifier les permissions (`sudo usermod -a -G dialout $USER`)
- Essayer un autre câble USB (certains câbles ne supportent que l'alimentation)

**Erreur de compilation avec PlatformIO :**
- Le projet utilise Arduino ESP32 3.x. Le fichier `platformio.ini` est configuré pour utiliser la bonne version.

**L'écran reste noir :**
- Vérifier que le téléversement s'est bien terminé
- Essayer de réinitialiser la carte (bouton RST)

## Licence

Voir les licences respectives des bibliothèques utilisées :
- LVGL : MIT License
- ESP32-audioI2S : GPL v3
