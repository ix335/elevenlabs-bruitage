---
name: elevenlabs-bruitage
description: Génère bruitages, foley, ambiances, whooshs, impacts et drones avec l'API Sound Effects d'ElevenLabs et les mixe autour de la voix. À utiliser pour tout son non vocal d'une vidéo.
---

# Bruitage et ambiances avec ElevenLabs

Ce skill complète `elevenlabs-voix-humaine`. La voix reste toujours la priorité : les sons servent la scène, ils ne la recouvrent pas.

> Les valeurs ci-dessous sont des points de départ. Ajuste-les selon la scène, avec un but :
> que l'auditeur croie à l'espace et à l'action sans jamais perdre un mot.

## 1. Découper avant de générer (cue sheet)
Lis le script ou la scène et dresse la liste des sons. Pour chacun, note :
- **Moment** : timecode ou phrase du texte où il tombe.
- **Fonction** : synchro / foley (pas, porte, objet), ambiance (fond continu), transition (whoosh), accent (impact, braam), tension (drone).
- **Durée**, **distance** (proche, moyen, lointain), **lieu** (pièce, extérieur, couloir, cave…).

Un fichier = un événement. Ne demande jamais une séquence (« il marche puis tombe dans l'escalier ») en un seul prompt : génère chaque son séparément et assemble-les au montage.

## 2. Appel API
```
POST https://api.elevenlabs.io/v1/sound-generation?output_format=mp3_44100_128
Header : xi-api-key: $ELEVENLABS_API_KEY
Body JSON :
  text              description du son
  model_id          "eleven_text_to_sound_v2"
  duration_seconds  0.5 à 30 (null = durée choisie par le modèle)
  prompt_influence  0 à 1 (défaut 0.3 ; haut = plus littéral)
  loop              true pour une boucle sans couture (modèle v2)
```
Utilise `pcm_44100` si le son doit encore être beaucoup traité.

Coût : en durée auto, 200 caractères par génération ; en durée fixée, 40 caractères par seconde. Fixe la durée pour les sons courts (c'est moins cher et plus précis).

### Réglages de départ par type
| Type | Durée | prompt_influence | loop |
|---|---|---|---|
| Foley / one-shot (porte, pas, objet) | 0,5–2 s | 0,6–0,8 | non |
| Son d'interface (UI) | ~1 s | 0,8 | non |
| Whoosh / transition | 1–2 s | 0,5–0,7 | non |
| Impact / braam | 1–4 s | 0,5–0,7 | non |
| Ambiance (pièce, ville, forêt, pluie) | 10–30 s | 0,3–0,5 | oui |
| Drone / tension | 10–30 s | 0,2–0,4 (plus créatif) | oui |

## 3. Écrire le prompt
Écris le prompt en anglais. Structure :
**source + matière** → **action / attaque** → **taille** → **distance / perspective** → **lieu / acoustique** → **ambiance émotionnelle** → **mots de format**.

Exemples :
- `Heavy wooden door slowly creaking open, old rusty hinges, close perspective, small stone room, eerie`
- `Single footstep on wet gravel, light sneaker, medium distance, outdoor, night, foley, isolated`
- `Distant city traffic ambience at night, light rain on window, muffled from inside an apartment, steady, no voices, no music`
- `Low ominous drone, slowly building tension, dark cinematic, sound designed`

Règles :
- Sois précis sur la matière, la taille et la distance. Un petit objet décrit « proche et grave » sonne industriel.
- Ajoute `no music, no voices` aux ambiances et foley si besoin. Une génération qui contient de la musique ou des voix est rejetée.
- Mots utiles : foley, whoosh, impact, braam, glitch, drone, one-shot, isolated, sound designed, close-up / distant, muffled, reverberant, dry.

## 4. Générer, choisir, monter
- Génère 2 ou 3 variantes. Si aucune ne convient, ne change qu'**une variable** à la fois (matière, distance, attaque) et note ce qui a changé.
- Écoute toujours dans le contexte, avec la voix, au niveau réel. Pas en solo.
- Coupe les blancs sans abîmer l'attaque. Place la transitoire **sur l'image exacte** de l'action, avec des fondus de 5–10 ms pour éviter les clics. Les pas peuvent arriver très légèrement en avance ; un clic d'interface en retard sonne faux.
- Préfère régénérer avec la bonne durée plutôt qu'étirer un son.
- Superpose par fonction : transitoire, poids (grave), matière, queue de pièce. Coupe chaque couche une à une ; si rien ne change, supprime-la.
- Varie les sons répétés : jamais deux fois le même pas d'affilée, jamais le même whoosh à chaque coupe.
- Boucles : seulement pour les sons continus. Vérifie plusieurs raccords (élément reconnaissable qui revient, pompage de niveau, saut stéréo, rythme visible). Deux boucles décalées l'une sur l'autre masquent la répétition.

## 5. Cohérence avec la voix
- **Hiérarchie** : voix → sons qui expliquent l'action → musique → ambiance → détails décoratifs.
- **Même espace** : la réverbération des sons suit le lieu de la scène, comme la voix (section 7.3 de `elevenlabs-voix-humaine`). Une voix sèche dans une cathédrale réverbérante ne tient pas.
- **Suivre le profil de voix** :
  - Narratif : ambiances discrètes, foley sobre, quelques accents aux transitions.
  - Chuchoté / intime : ambiance très basse, sons petits et proches, presque rien d'autre.
  - Tension / panique : sons plus secs et rapides, battements, respirations, drone qui monte.
  - Cri : les impacts peuvent monter fort, mais ils ne tombent jamais sur un mot.
- **Ne pas masquer la parole** : pendant que la voix parle, creuse légèrement l'ambiance vers 2–5 kHz et baisse un son seulement là où il cache des mots. Automatise le niveau avant d'empiler du traitement.

## 6. Niveaux (points de départ, relatifs à la voix)
| Élément | Sous la voix | Sans voix |
|---|---|---|
| Ambiance | −15 à −25 dB | peut remonter, reste un fond |
| Foley | −6 à −12 dB | naturel, logique physique de la scène |
| Whoosh / transition | −8 à −12 dB | jusqu'au niveau de la voix |
| Impact / braam | à éviter sur un mot | jusqu'au niveau de la voix, pics contenus |
| Drone | −18 à −25 dB | monte avec la tension |

Traitement : passe-haut sur le grondement inutile (sauf impacts et drones qui ont besoin de grave), coupe étroite sur les bandes dures, compression légère. La compression ne sauve pas un mauvais son : régénère-le.

Niveau final : normalise le mix complet (voix + musique + sons) en une fois, selon la section 7.5 de `elevenlabs-voix-humaine`. Pics sous −1 dBTP.

## 7. Vérifier avant de livrer
- Aucune musique ni voix parasite dans les sons.
- Chaque son tombe sur l'image ; pas de clic aux coupes.
- Les boucles ne laissent pas entendre leur raccord.
- La voix reste intelligible partout, y compris pendant les impacts.
- Les sons répétés varient.
- Niveau intégré et pics conformes.
