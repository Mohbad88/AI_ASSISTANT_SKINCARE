# Assistant IA Pour protection de Peau

AI Assisant Skincare est un assistant IA de consultation basé sur Gradio. Il accepte une description vocale du patient ainsi qu’une image ou une vidéo de la peau, transcrit l’audio, transmet le contexte visuel et textuel à un modèle d’IA, puis fournit une réponse écrite et audio dans un style médical.

L’application utilise actuellement :

- Gradio pour l’interface Web.
- Groq Whisper pour la transcription vocale du patient.
- Le modèle de vision de Groq pour fournir des informations sur la peau à partir d’images.
- Deepgram Text-to-Speech pour générer la réponse audio du médecin.
- `uv` pour gérer la version de Python, l’environnement virtuel, les dépendances et le fichier de verrouillage.

Important : ce projet fournit uniquement des informations générales. Il ne constitue pas un diagnostic médical et ne remplace pas les conseils ou les soins d’un dermatologue ou d’un professionnel de santé agréé.

## Structure du projet

```text
AI_Assistant_Skincare/
|-- main.py                     # Point d’entrée de l’application Gradio
|-- voice_of_the_patient.py     # Enregistrement au microphone et transcription Groq
|-- brain_of_the_doctor_groq.py # Génération de réponses texte/vision avec Groq utilisée par main.py
|-- brain_of_the_doctor.py      # Implémentation alternative compatible avec MiniMax/Anthropic
|-- voice_of_the_doctor.py      # Génération audio text-to-speech avec Deepgram
|-- free_text_to_speech.py      # Autre expérimentation/outil de synthèse vocale
|-- sample.env                  # Modèle de variables d’environnement
|-- pyproject.toml              # Métadonnées du paquet Python et dépendances
|-- uv.lock                     # Versions verrouillées des dépendances générées par uv
|-- .python-version             # Version de Python demandée par uv et les outils similaires
`-- README.md
```

## Prérequis

- Python 3.11 ou ultérieur.
- Le gestionnaire de paquets `uv`.
- FFmpeg pour la conversion et la lecture audio utilisées par `pydub` et les outils multimédias.
- PortAudio pour la prise en charge du microphone et de l’audio utilisée par `pyaudio` et `speech_recognition`.
- Des clés API Groq et Deepgram.

## Installer les dépendances système

Installez FFmpeg et PortAudio avant de synchroniser les dépendances Python. Cela permet d’éviter les problèmes courants liés à l’installation de `pyaudio`, au microphone et à la conversion audio.

### macOS

Avec Homebrew :

```bash
brew update
brew install ffmpeg portaudio
```

Vérifiez l’installation :

```bash
ffmpeg -version
brew list portaudio
```

Si la compilation de `pyaudio` échoue, vérifiez que PortAudio installé par Homebrew est accessible :

```bash
brew --prefix portaudio
```

Sur les Mac avec puce Apple Silicon, Homebrew se trouve généralement dans `/opt/homebrew`. Sur les Mac Intel, il se trouve généralement dans `/usr/local`.

### Windows

Option recommandée avec Chocolatey, dans PowerShell exécuté en tant qu’administrateur :

```powershell
choco install ffmpeg portaudio
```

Autre possibilité avec Scoop :

```powershell
scoop install ffmpeg
scoop install portaudio
```

Vérifiez FFmpeg :

```powershell
ffmpeg -version
```

Remarques concernant Windows :

- Redémarrez le terminal après l’installation de FFmpeg afin que les changements de `PATH` soient pris en compte.
- Si l’installation de `pyaudio` échoue, installez Microsoft C++ Build Tools depuis `https://visualstudio.microsoft.com/visual-cpp-build-tools/`, puis relancez `uv sync`.
- Certaines configurations Windows peuvent installer automatiquement une version précompilée de `pyaudio`. Sinon, PortAudio et les outils de compilation C++ sont généralement les éléments manquants.

### Linux

Ubuntu/Debian :

```bash
sudo apt update
sudo apt install ffmpeg portaudio19-dev python3-dev build-essential
```

Fedora :

```bash
sudo dnf install ffmpeg portaudio-devel python3-devel gcc gcc-c++ make
```

Arch Linux :

```bash
sudo pacman -S ffmpeg portaudio base-devel
```

Vérifiez l’installation :

```bash
ffmpeg -version
```

Si le microphone ne fonctionne pas sous Linux, vérifiez également que votre système audio fonctionne et que l’utilisateur actuel a accès au périphérique d’entrée.

## Installer uv

`uv` est la méthode recommandée pour créer l’environnement virtuel, installer les dépendances et lancer l’application.

### macOS et Linux

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Redémarrez votre terminal ou chargez la mise à jour de l’environnement affichée par le programme d’installation. Vérifiez ensuite :

```bash
uv --version
```

### Windows

Dans PowerShell :

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Redémarrez PowerShell, puis vérifiez :

```powershell
uv --version
```

D’autres méthodes d’installation sont également disponibles :

```bash
pipx install uv
```

ou :

```bash
pip install uv
```

## Configuration

Clonez le projet ou ouvrez son dossier, puis exécutez :

```bash
uv sync
```

Cette commande lit `pyproject.toml` et `uv.lock`, crée `.venv` si nécessaire, installe Python 3.11 si votre configuration `uv` prend en charge le téléchargement de versions gérées, puis installe toutes les dépendances verrouillées.

Pour forcer l’utilisation de Python 3.11 :

```bash
uv python install 3.11
uv sync --python 3.11
```

## Variables d’environnement

Créez un fichier `.env` à la racine du projet. Vous pouvez le copier depuis le modèle fourni :

```bash
cp sample.env .env
```

Renseignez ensuite les valeurs :

```bash
GROQ_API_KEY=your_groq_api_key_here
DEEPGRAM_API_KEY=your_deepgram_api_key_here

# Remplacements facultatifs des modèles
WHISPER_MODEL=whisper-large-v3
GROQ_MODEL=meta-llama/llama-4-scout-17b-16e-instruct
DEEPGRAM_TTS_MODEL=aura-2-thalia-en

# Facultatif : uniquement si vous utilisez brain_of_the_doctor.py plutôt que brain_of_the_doctor_groq.py
MINIMAX_API_KEY=your_minimax_api_key_here
MINIMAX_BASE_URL=https://api.minimax.io/anthropic
MINIMAX_MODEL=MiniMax-M3
```

Variables requises pour le fonctionnement par défaut de `main.py` :

- `GROQ_API_KEY` pour la transcription audio et l’analyse d’images.
- `DEEPGRAM_API_KEY` pour générer la réponse vocale du médecin.

Ne versionnez pas de vraies clés API.

## Lancer l’application

Utilisez `uv run` pour exécuter la commande dans l’environnement du projet :

```bash
uv run python main.py
```

Gradio affichera une URL locale, généralement :

```text
http://127.0.0.1:7860
```

Ouvrez cette URL dans votre navigateur.

## Utilisation

1. Enregistrez ou importez une description vocale du patient.
2. Importez une image de la peau. L’implémentation Groq actuelle exige une image.
3. Vous pouvez également importer une vidéo de la peau. L’implémentation Groq actuelle ne traite pas directement les vidéos et utilise l’image comme référence visuelle.
4. Cliquez sur `Analyze Concern` (« Analyser le problème »).
5. Consultez la transcription, les conseils générés et la réponse audio.

## Commandes uv

Commandes courantes pour ce projet :

```bash
# Créer/mettre à jour l’environnement virtuel à partir de pyproject.toml et uv.lock
uv sync

# Lancer l’application Gradio
uv run python main.py

# Exécuter un module ou un script individuellement
uv run python voice_of_the_patient.py

# Ajouter une dépendance
uv add package-name

# Ajouter une dépendance de développement
uv add --dev package-name

# Supprimer une dépendance
uv remove package-name

# Mettre à jour le fichier de verrouillage sans installer les dépendances
uv lock

# Mettre à niveau les dépendances dans les limites de version autorisées
uv lock --upgrade

# Afficher les paquets installés dans l’environnement
uv pip list

# Afficher l’arbre des dépendances
uv tree
```

En général, il n’est pas nécessaire d’activer manuellement l’environnement virtuel avec `uv run`. Si vous souhaitez tout de même l’activer :

macOS/Linux :

```bash
source .venv/bin/activate
```

Windows PowerShell :

```powershell
.venv\Scripts\Activate.ps1
```

Windows Command Prompt :

```cmd
.venv\Scripts\activate.bat
```

Pour le désactiver :

```bash
deactivate
```

## Dépendances Python

Les principales dépendances sont répertoriées dans `pyproject.toml` :

- `gradio` pour l’application Web.
- `groq` pour Whisper et le modèle de vision/conversation.
- `deepgram-sdk` pour la synthèse vocale.
- `anthropic` pour l’implémentation alternative compatible avec MiniMax.
- `python-dotenv` pour charger les variables du fichier `.env`.
- `pydub`, `pyaudio`, `sounddevice` et `speechrecognition` pour l’enregistrement et la conversion audio.
- `pillow` pour redimensionner et encoder les images avant leur envoi à Groq.
- `numpy` et `path` comme dépendances complémentaires.

Puisque ce projet inclut `uv.lock`, privilégiez `uv sync` plutôt que `pip install -r requirements.txt`. Le projet ne contient pas de fichier `requirements.txt`.

## Notes de développement

- `main.py` importe actuellement `brain_of_the_doctor` depuis `brain_of_the_doctor_groq.py`.
- `brain_of_the_doctor_groq.py` exige une image en entrée. Si seule une vidéo est importée, la validation de l’application réussit, mais Groq Vision renvoie une erreur puisqu’aucune image n’a été fournie.
- `brain_of_the_doctor.py` contient une méthode alternative compatible avec MiniMax/Anthropic, capable d’encoder une image ou une vidéo, mais elle n’est pas importée actuellement dans `main.py`.
- Par défaut, l’audio généré est enregistré dans `doctor_response.mp3`.
- Dans `main.py`, Gradio est lancé avec `debug=True`.

## Dépannage

### `ffmpeg` introuvable

Installez FFmpeg en suivant les instructions correspondant à votre système d’exploitation, redémarrez votre terminal et vérifiez l’installation :

```bash
ffmpeg -version
```

### Échec de l’installation de `pyaudio`

Cela signifie généralement que les en-têtes de PortAudio ou les outils de compilation sont manquants.

- macOS : exécutez `brew install portaudio`.
- Windows : installez PortAudio et Microsoft C++ Build Tools, puis relancez `uv sync`.
- Ubuntu/Debian : exécutez `sudo apt install portaudio19-dev python3-dev build-essential`.
- Fedora : exécutez `sudo dnf install portaudio-devel python3-devel gcc gcc-c++ make`.
- Arch : exécutez `sudo pacman -S portaudio base-devel`.

### Microphone non détecté

- Vérifiez les autorisations du microphone dans votre système d’exploitation.
- Fermez les autres applications susceptibles d’utiliser le microphone.
- Vérifiez que votre navigateur est autorisé à utiliser le microphone pour l’URL locale de Gradio.
- Vérifiez que PortAudio est installé.

### Erreurs de clé API manquante

Assurez-vous que le fichier `.env` existe à la racine du projet et contient au minimum :

```bash
GROQ_API_KEY=your_groq_api_key_here
DEEPGRAM_API_KEY=your_deepgram_api_key_here
```

Lancez ensuite l’application depuis la racine du projet :

```bash
uv run python main.py
```

### Échec de la génération audio Deepgram

Vérifiez que `DEEPGRAM_API_KEY` est valide et que le modèle `DEEPGRAM_TTS_MODEL` sélectionné est disponible pour votre compte.

### Échec de l’analyse d’image Groq

Vérifiez que `GROQ_API_KEY` est valide et que `GROQ_MODEL` prend en charge les images. L’implémentation par défaut convertit l’image importée en JPEG, la redimensionne pour qu’elle tienne dans une zone de `1024x1024`, puis l’envoie sous forme d’URL de données encodée en base64.

## Sécurité

- Gardez le fichier `.env` privé.
- Ne versionnez pas de vraies clés API.
- Les images, vidéos et fichiers audio des patients doivent être traités comme des données sensibles.
- Ce dépôt inclut des fichiers multimédias d’exemple pour les tests locaux. Pour une utilisation en production, définissez une politique claire de conservation et de confidentialité des données.

## Avertissement médical

Cette application est un assistant d’information générale sur les soins de la peau. Elle ne peut ni diagnostiquer une maladie, ni prescrire un traitement, ni remplacer un professionnel de santé. En cas de symptômes graves, d’aggravation rapide, de fièvre, de douleur, de saignement, de signes d’infection ou de toute autre urgence, contactez immédiatement un professionnel de santé agréé.
