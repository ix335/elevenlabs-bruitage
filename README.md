# elevenlabs-bruitage

Skill Claude pour générer des bruitages, du foley, des ambiances, des whooshs, des impacts et des drones avec l'API Sound Effects d'ElevenLabs (`eleven_text_to_sound_v2`), puis les mixer autour de la voix.

Il complète [human-voice-elevenlab](https://github.com/ix335/human-voice-elevenlab) : mêmes profils de scène (narratif, chuchoté, panique, cri), même logique d'espace et une seule normalisation finale du mix.

## Ce que fait le skill
- Découpe la scène en liste de sons (cue sheet) : un fichier par événement.
- Choisit durée, `prompt_influence` et `loop` selon le type de son.
- Écrit des prompts précis (matière, taille, distance, lieu, émotion).
- Génère des variantes, cale chaque son sur l'image, superpose par fonction.
- Garde la voix toujours intelligible et place les sons dans le même espace qu'elle.

## Installation
Copie le dossier `skills/elevenlabs-bruitage/` dans tes skills Claude.
Il faut une clé API ElevenLabs (`ELEVENLABS_API_KEY`).

## Licence
MIT
