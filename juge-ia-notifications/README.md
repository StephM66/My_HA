# Juge IA des notifications pour Home Assistant

Vos automatisations ne notifient plus directement : elles décrivent l'événement à un **juge IA**, qui décide s'il faut prévenir, et comment. Notification mobile, annonce vocale, ou simple ligne dans le journal du jour.

L'IA ne pilote rien : elle donne un avis en JSON, et le script applique des garde-fous.

- **IA en panne ou réponse illisible** : le message de secours part quand même.
- **Confiance faible** : notification discrète plutôt que rien.
- **Voix impossible** (personne à la maison, quelqu'un dort) : mobile à la place. Seule une urgence critique parle la nuit.
- **Tout est noté** dans une liste de tâches.

Le contexte de la maison est construit **automatiquement** : personnes et zones, sommeil, média en cours, agenda, et toute entité portant l'étiquette « Contexte IA ».

## Installation rapide : les blueprints (recommandé)

[![Importer le juge](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FStephM66%2FMy_HA%2Fblob%2Fmain%2Fjuge-ia-notifications%2Fblueprints%2Fscript%2Fjuge_ia_notification.yaml) **Juge** : décide, puis diffuse par les canaux choisis (voix, téléphones, Telegram…), chacun avec son repli.

[![Importer le débrief](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FStephM66%2FMy_HA%2Fblob%2Fmain%2Fjuge-ia-notifications%2Fblueprints%2Fscript%2Fdebrief_ia.yaml) **Débrief** : raconte ce que le juge a gardé pour plus tard, puis vide le journal.

1. Créez un script depuis le blueprint juge, nommé **« IA – Juge de notification »** (`script.ia_juge_de_notification`).
2. Créez un script depuis le blueprint débrief, nommé **« IA – Débrief »** (`script.ia_debrief`).
3. Ajoutez [`automatisations/debrief_soir.yaml`](automatisations/debrief_soir.yaml) et/ou [`automatisations/debrief_retour.yaml`](automatisations/debrief_retour.yaml).

Modes du juge : `juge` (l'IA décide), `journal` (noter sans notifier), `diffuser` (envoyer directement, sans IA). Votre propre script d'annonce peut servir de canal vocal : voir [`exemples/script_annonce_perso.yaml`](exemples/script_annonce_perso.yaml).

Les sections suivantes décrivent la version script simple, sans blueprint.

## Contenu

| Fichier | Rôle |
|---|---|
| [`script/ia_juge_de_notification.yaml`](script/ia_juge_de_notification.yaml) | le juge (script) |
| [`automatisations/porte.yaml`](automatisations/porte.yaml) | porte d'entrée restée ouverte |
| [`automatisations/lave_linge.yaml`](automatisations/lave_linge.yaml) | lave-linge terminé |
| [`automatisations/fuite.yaml`](automatisations/fuite.yaml) | fuite d'eau (urgence) |
| [`automatisations/rendez_vous.yaml`](automatisations/rendez_vous.yaml) | rappel une heure avant un événement d'agenda |
| [`automatisations/batteries.yaml`](automatisations/batteries.yaml) | piles faibles notées au journal chaque soir, sans IA |
| [`blueprints/script/juge_ia_notification.yaml`](blueprints/script/juge_ia_notification.yaml) | blueprint juge (canaux et replis) |
| [`blueprints/script/debrief_ia.yaml`](blueprints/script/debrief_ia.yaml) | blueprint débrief |
| [`automatisations/debrief_soir.yaml`](automatisations/debrief_soir.yaml) | débrief chaque soir |
| [`automatisations/debrief_retour.yaml`](automatisations/debrief_retour.yaml) | débrief au retour à la maison |
| [`exemples/script_annonce_perso.yaml`](exemples/script_annonce_perso.yaml) | script d'annonce compatible avec le juge |
| [`exemples/capteur_quelqu_un_dort.jinja`](exemples/capteur_quelqu_un_dort.jinja) | modèle d'un capteur « Quelqu'un dort » |

## Prérequis

- Home Assistant **2025.8 ou plus récent** (action `ai_task.generate_data`).
- Une entité **AI Task** : OpenAI (gpt-4o-mini avec les paramètres recommandés), ou une IA locale comme Ollama.
- L'application mobile Home Assistant.
- Une liste de tâches, par exemple « Liste de tâches locale » nommée « Débrief maison ».
- Facultatif : un moteur de synthèse vocale et une enceinte.

## Installation

1. **Paramètres › Automatisations et scènes › Scripts › Créer un script › ⋮ › Modifier en YAML**, puis collez [`script/ia_juge_de_notification.yaml`](script/ia_juge_de_notification.yaml). Gardez l'identifiant `ia_juge_de_notification`.
2. Adaptez le bloc **« Réglages de ta maison »**, en tête de la séquence :

   | Réglage | Rôle | Exemple |
   |---|---|---|
   | `ia` | entité AI Task | `ai_task.openai_ai_task` |
   | `notifier_mobile` | téléphone | `notify.mobile_app_mon_telephone` |
   | `liste_debrief` | journal | `todo.debrief_maison` |
   | `lits` | capteurs « quelqu'un dort » (`[]` si aucun) | `[binary_sensor.lit_parents]` |
   | `lecteurs` | lecture = film ou musique en cours (`[]` si aucun) | `[media_player.tv_salon]` |
   | `agenda` | agenda à consulter (`""` si aucun) | `calendar.famille` |
   | `agenda_heures` | heures d'agenda envoyées | `12` |
   | `etiquette_contexte` | étiquette des entités envoyées | `Contexte IA` |
   | `tts_moteur`, `tts_lecteur` | annonce vocale (`""` pour ne pas en faire) | `tts.google_translate_fr_fr`, `media_player.cuisine` |

3. Dans **Paramètres › Entités**, ajoutez l'étiquette **« Contexte IA »** aux entités utiles : alarme, température extérieure, mode invités, vacances…
4. Créez une automatisation à partir d'un exemple (**⋮ › Modifier en YAML**) et remplacez les entités par les vôtres.
5. Testez : **Outils de développement › Actions › `script.ia_juge_de_notification`**. La trace du script montre le contexte envoyé et la réponse de l'IA.

## Appeler le juge

```yaml
action: script.ia_juge_de_notification
data:
  message: Le lave-linge a terminé.          # envoyé tel quel si l'IA ne répond pas
  contexte: Jamais urgent.                   # ce que l'IA ne peut pas deviner
  mode_juge: juge                            # ou « journal » : noter sans notifier ni appeler l'IA
```

## Coût (OpenAI, gpt-4o-mini)

Environ 900 jetons en entrée et 120 en sortie par appel, soit environ **0,0002 $ par appel** et **0,06 $ par mois pour 10 notifications par jour**. Ce calcul utilise le tarif standard de gpt-4o-mini : 0,15 $ par million de jetons en entrée et 0,60 $ en sortie. Vérifiez la page tarifs d'OpenAI : les prix et les modèles disponibles évoluent. Le mode `journal` n'appelle pas l'IA.

## Vie privée

À chaque appel, le fournisseur d'IA reçoit plusieurs informations :

- le message et le contexte de l'automatisation ;
- les noms des personnes et des zones ;
- les titres de l'agenda ;
- l'état des entités étiquetées.

N'étiquetez jamais de code, de mot de passe ou de caméra. Pour que rien ne sorte de chez vous, utilisez une IA locale.

## Licence

MIT (voir [LICENSE](../LICENSE)). Retours et améliorations bienvenus via les *issues*.
