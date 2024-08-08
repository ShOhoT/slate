---
title: API Reference

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - shell
  # - python
  - markdown

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Kardinal API
---

# Informations préambules

## Principe général

L’API est axée autour de deux éléments principaux : 

- l’envoi de plans : les données du problèmes
- la récupération de solutions : le plan de tournées proposé par l’algorithme

```markdown
  Dans le fichier xlsx, le champ se trouve dans l'onglet `xxx` colonne `yyy`
```

## Optimisation Continue

Nos algorithmes produisent des solutions en continue : de plus en plus pertinentes. L’intérêt est d’avoir accès de manière plus souples à la meilleure solution trouvée à un instant t.

L’algo ne peut publier une solution moins bonne que la précédente.

## Moteur d’optimisation intéractif

Nos algorithms sont conçus pour s’adapter rapidement aux changements dans le plan.

Pour indiquer à l’algorithme que des modifications ont lieu sur le même problème, il faut fournir le même identifiant de plan dans la demande de mise à jour.

Le champ version du plan sera incrémenté.

Comme les moteurs produisent des solutions en continue, ils vont produire des solutions de la version précédente avant de produire des solutions de la dernière version.

Le moteur est prévu pour dégrader sa solution et trouver rapidement une nouvelle solution valide.

## Formats

ARO accepte deux formats d’entrée pour le `plan` et fourni deux formats sortie pour la `solution` :

- Xlsx
- JSON

<aside>
💡 **Les noms de colonnes du fichier xlsx sont identiques aux noms des variables du fichier json**

</aside>

 Transmis soit 

- [par l’API](https://aro.kardinal.ai/api/v2/doc/openapi.html#/Plan/putPlan)
- soit par notre front-end (dans la vue “+”)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/b4899a30-c50e-4ea1-8578-94d05bf9aa44/579325ed-a2f1-4992-bb4a-f628a08a0934/Untitled.png)

## API sans états

Il est conseillé d’utiliser majoritairement l’endpoint [de mise à jour du plan](https://plt.dev.kardinal.ai/api/v2/doc/openapi.html#/Plan/putPlan) : il suffit de transformer l’ensemble des données concernées par l’optimisation à chaque fois et de les fournir à Kardinal.

Il existe des endpoints unitaire d’ajout d’ordre ou de retrait de ressources, mais ils impliquent de conserver **un état intermédiaire** (savoir ce qui a été transmis exactement à Kardinal) ce qui est plus lourd en termes d’intégration.

## Quotas d’optimisation

Les quotas d’optimisation définissent les droits d’utilisations : 

- nombre de plans en parallèle
- nombre de ressources par plan
- nombre de stops par plan
- nombre d’ordres par plan
- nombre des stops par ordre (non successifs)

Ils nous permettent d’éviter des facturations non prévues.

# Authentification

> **Linux / MacOS**

```bash
curl -X POST "<env>/api/v2/login" \
-H "Accept: application/json" \ 
-H "Content-Type: application/json" \ 
-d '{"password":"<password>","username":"<username>"}'
```

> **Windows**

```powershell
Invoke-WebRequest -Uri "<env>/api/v2/login" -Method POST -Headers @{"Accept"="application/json"; "Content-Type"="application/json"} -Body '{"password":"<password>","username":"<username>"}'
```

Le système d’authentification basé sur le protocole JWT.

Le jeton d’accès, ou `access_token` , est généré lors du login (via login/mot de passe ou sso).

<aside>
  env et username sont fournis dans l'invitation mail, qui invite à configurer votre mot de passe
</aside>

> Réponse (succès)

```json
{
	"access_token":"<token>",
	"refresh_token":"<refresh_token>",
	"user":{ ... }
}
```

En cas de succès, vous recevrez le jeton d’accès `access_token`.

En cas d’echec, vous recevrez une erreur.

Avec ce jeton, vous avez tout ce qu’il faut pour utiliser l’API. Le jeton est valide 1h et peut-être rafraichit via la méthode `login` ou la méthode `refreshToken`

# Plan

| Champ | Description | Obligatoire | XLSX  |
|---|---|---|---|
|   |   |   |   |
|   |   |   |   |
|   |   |   |   |

## maxOptimizationDuration

Iso8601 Duration

Optional

Default value = `quota` default value.

La durée maximale d’optimisation définie le temps que le moteur va allouer à l’optimisation du plan (sans compter la transformation et la récupération des données de trajet).

## tz

Timezone as String

Optional.  

Default value = “Europe/Paris”

Permet d’indiquer la timezone attendue dans l’échange de plan et de solution.

## objectives

Array of String

Optional

Default value : 

```json
"objectives": [
  "maximizeMandatoryStops",
  "minimizeDelay",
  "minimizeCosts",
  "minimizeResources",
  "minimizeOverOverlappingCapacitiesOnStops",
  "maximizeOptionalStops",
  "maximizePreferredStops",
  "minimizeWorkingDuration",
  "minimizeDistance"
],
```

Les objectifs permettent de définir comment l’algorithme doit se comporter pour respecter les priorités business.

Par défaut, les objectifs ne sont pas utilisés s’ils ne sont pas pertinents : 

- S’il n’y a pas de `preferredTimeWindows` il n’y a pas de `minimizeDelay`
- S’il n’y a pas de coûts, il n’y a pas `minimizeCosts`
- S’il n’y a pas de `overlappingCapacitiesByStopTag`, il n’y a pas de `minimizeOverOverlappingCapacitiesOnStops`
- S’il n’y a pas de `preferredStopTags` il n’y a pas de `maximizePreferredStops`

L’ordre par défaut fourni permet de résoudre de manière optimale 99 % des plans.

Nous recommandons fortement de conserver l’ordre fourni par défaut, mais c’est paramétrable.

<aside>
⚠️ Modifier les objectifs peuvent retourner des solutions difficile à appréhender pour les humains, notamment si l’on positionne trop haut l’objectif de distance, l’algorithme risque de faire des tournées très hétérogènes.

</aside>

Il est possible de faire coller les objectifs aux réalités business dans certains cas, par exemple :

- j’ai une flotte en propre, il faut qu’ils sortent tous : retirer l’objectif `minimizeResources`
- je souhaite mettre l’accent sur le CO2 : il est possible de remonter l’objectif `minimizeDistance`

# Order

Un order définie une suite de points à visiter par la même ressource, dans l’ordre fourni.

Par exemple : s’il faut enlever un meuble à un point A pour le livrer à un point B, cela correspond à un order avec deux stops : A (`kind = pickup`) et B (`kind = enlèvement`).

## Cas limites

Si l’on parle de charger son camion point de départ pour livrer des colis chez des clients, soit une tournée de livraison, il n’est pas utile de modéliser le chargement au dépôt. (1 seul stop)

Si l’on parle de récupérer des éléments chez des clients pour les ramener au point de départ, soit une tournée de collecte, il n’est pas utile de modéliser le retour au dépôt. (1 seul stop)

Il est possible de mélanger ces cas limites avec des enlèvement+livraison.

## id

String

Mandatory

No default value

Permet d’indiquer ce qu’on veut : un identifiant un interne pour le mapping, le nom du client, un mix identifiant et date… le fait d’avoir un champ identifiant libre permet beaucoup de simplifications.

Par contre il doit être unique pour chaque ordre.

## properties

String, key-value

Optional

No default value

Les [properties](https://www.notion.so/ARO-Functional-Documentation-4c14397d1bc344e498de31691b22e011?pvs=21) sont dans champs libres de type clé-valeur. Ils vous permettent de passer des paramètres importants à votre compréhension ou intégration.

## priority

Integer, [-1000,1000]

Optional

Default value = 0

Plus la priorité est basse, plus elle est considérée comme importante. Car il faut lire P0 (priorité = 0) plus prioritaire que P3 (priorité = 3).

Le moteur retirera n’importe quelle quantité d’ordre de priorité importante pour des priorités plus faibles.

Il existe plusieurs fonctions pour donner une priorité

## optional

Boolean

Optional

Default value = `false`

Optional est une forme de priorité. Elle indique qu’un ordre n’est pas critique au succès du plan.

## requiredSkills

Array of Strings

Optional

No default value

Les `requiredSkills` permettent d’identifier des attributs ou compétences nécessaires à la réalisation des ordres. Les resources possèdent elles aussi un champ `skills`qui indique les attributs ou compétences à leur disposition et qui sont corrélées.

Un exemple classique : les livraisons nécessitant un camion hayon.

Afin de réaliser un ordre, une ressource doit disposer de **la totalité** des `requiredSkills`.

Par exemple :

```json
{
	"id":"ordre-1",
	"requiredSkills":[
		"COMPTER_JUSQUA_10",
		"COMPTER_JUSQUA_30"
	]
}
```

Pour compter jusqu’à 30, il faut savoir compter jusqu’à 10. Inversement on ne sait pas forcément compter jusqu’à 30 (il faut imaginer…).

## successiveStops

Boolean

Optional

Default Value = `false`

Cette contrainte permet d’indiquer que les stops de l’ordre doivent être effectués à la suite, sans aucun ordre/stop intermédiaire. 

C’est pratique dans le cas où les contenants ne peuvent être mélangés par exemple.

Ca peut aussi être utilisé pour conserver une suite de tâche.

## maxStopSpan

Iso8601 Duration (String)

Optional

No default value

Cette contrainte ne s’utilise que dans un contexte de Pickup and Delivery.

Elle permet de modéliser le temps maximal pour réaliser la livraison à compter de l’enlèvement.

C’est très utilisé pour modéliser des durées de vie ou du transport de type taxi.

## stops

Stop

Mandatory (au moins 1)

No default value

Les stops d’un ordre sont à réaliser dans l’ordre fourni, par une seule et même ressource.

Il est en général conseillé de ramener ses problèmes à un stop (type tournée de collecte ou tournée de livraison ou tournée d’interventions) pour des soucis de performances.

Mais il est possible de modéliser des ordres avec plus de deux stops.

Voir le détail sur la documentation d’un Stop

# Stop

Les stops peuvent être de deux sortes

- `single` : un stop simple
- `alternatives` : le moteur doit choisir parmi la liste fournie le meilleur stop à visiter, c’est une liste de `single` stops

Ce variation est renseignée par le champ `type`.

Les `alternatives` sont utilisés quand :

- vous devez choisir parmi plusieurs emplacement pour jeter vos déchets (incinérateurs, déchetteries)
- vous devez trouver un point de recharge optimal
- vous pouvez vous ravitailler en cargaison à plusieurs endroits

<aside>
💡 Attention : les `alternatives` peuvent être coûteux en temps d’optimisation.

</aside>

## id

String

Mandatory

Default value = `single`

Permet d’indiquer ce qu’on veut : un identifiant un interne pour le mapping, le nom du magasin…

Le fait d’avoir un champ identifiant libre permet beaucoup de simplifications.

Par contre il doit être unique pour chaque stop.

## properties

String, key-value

Optional

No default value

Les [properties](https://www.notion.so/ARO-Functional-Documentation-4c14397d1bc344e498de31691b22e011?pvs=21) sont dans champs libres de type clé-valeur. Ils vous permettent de passer des paramètres importants à votre compréhension ou intégration.

## position

Coordinates

Mandatory

No default value

La position géographiques / coordonnées du stop.

Latitude (lat) et longitude (lon).

```json
{
  "lon": 2.3269331,
  "lat": 48.8812658
}
```

Nécessite d’avoir réalisé le géocodage des adresses.

## kind

Enum (`pickup`, `delivery`, `acknowledgement` )

Optional

Default value = `pickup`

Indique le type du stop, déclenche des choses côté algorithme : 

- s’il existe une capacité sur le stop et que le `kind` est `delivery`, alors la `capacity` consommée est retirée de la ressource
    - dans le cas d’une tournée de livraison, la `capacity` de chaque stop est utilisée pour calculer le remplissage du camion au départ
- s’il existe une capacité sur le stop et que le `kind` est `pickup`, alors la `capacity` doit pouvoir être ajoutée à la resource
- si le `kind` est `acknowledgement`, la `capacity`n’est pas considéré. Ce type est utilisé pour modéliser des interventions sans échanges de cargaison.

## operationDuration

Duration ISO8601

Optional

Default value = `PT0M` 

Indique la durée nécessaire à la réalisation de l’opération une fois arrivé sur le lieu.

Cette valeur à un impact fort sur l’optimisation et doit être déterminée au plus proche de la réalité.

Il est possible de définir des durée opératoires différentes pour chaque stops ou d’appliquer des stratégies de durée opératoire groupées côté intégrateur.

## capacities

Key-value map

Optional

No default value

Les capacities permettent d’indiquer des contraintes liées aux marchandises. Le format de ce champ est libre pour permettre de déclarer tout type de capacité.

Dans le cas d’un stop, capacities indique ce que le chargement va occuper.

La seule chose requise est de retrouver les mêmes valeurs sur au moins une ressource dans le plan, afin que l’algorithme puisse affecter cet ordre aux ressources éligibles.

Par exemple, pour un chargement multi-température :

- poids à température ambiante : `{ "weight":33.0 }`
- poids dans bac frigorifique : `{ "freezer": 12.0 }`

## timeWindows

begin : ISO8601

end : ISO8601

Un créneau horaire est composé d’une date et heure de début et d’une date et heure de fin.

L’algorithme fera tout seul l’intersection entre tous les créneaux authorisés et préférés. 

### authorizedTimeWindows

Array of TimeWindow 

Optional

No default value

Les fenêtres de temps autorisées décrivent un créneau horaire que l’algorithme doit respecter absolument. Si le créneau ne peut être respecté, l’algorithme considère que le créneau ne peut être respecté.

Les solutions ne peuvent contenir de créneaux horaires qui ne sont pas respectés.

Elles permettent de modéliser des engagements forts avec les clients finaux notamment.

Lorsque l’on applique des créneaux horaires, il est important de les faire aussi large que possible.

Si l’on prend rendez-vous pour 15h, on peut considérer qu’arriver en avance et attendre est acceptable, et souvent tolérer 5 minutes de retard.

Donc la fenêtre serait : 

```json
{
	"begin":"YYYY-MM-DDT14:50:00Z",
	"end":"YYYY-MM-DDT15:05:00Z",
}
```

Il est important de créer les créneaux autorisées les plus larges possibles afin de laisser plus de libértés et d’optimisation au moteur.

Il est possible de modéliser des créneaux d’ouvertures aussi, via plusieurs time windows.

Par exemple, si un magasin est ouvert entre 8:00 et 12:00, puis 14:00 à 19:00 :

```json
[
	{
		"begin":"YYYY-MM-DDT08:00:00Z",
		"end":"YYYY-MM-DDT12:00:00Z",
	},
	{
		"begin":"YYYY-MM-DDT14:00:00Z",
		"end":"YYYY-MM-DDT19:00:00Z",
	}
]
```

<aside>
⚠️ Par défaut, l’algorithme considère qu’arriver sur le point au dernier moment pour y effectuer la tâche est possible. Si l’on veut effectuer la tâche avant la fermeture, il faut retrancher la durée opératoire de la borne `end`

</aside>

### preferredTimeWindows

Array of TimeWindow

Optional

No default value

Les fenêtres de temps préferentielles décrivent des créneaux horaires que l’algorithme doit respecter **si possible**. 

Si le créneau ne peut être respecté, l’algorithme considère qu’il accumule du retard.

Le retard est un des objectifs principaux de l’optimisation.

Ces créneaux sont pratiques pour modéliser des souhaits du clients qui ne seraient pas contractuellement pénalisant.

# Resources

## id

String

Mandatory

Default value = `single`

Permet d’indiquer ce qu’on veut : un identifiant un interne pour le mapping, le nom du chauffeur…

Le fait d’avoir un champ identifiant libre permet beaucoup de simplifications.

Par contre il doit être unique pour chaque ressource.

## priority

Integer, [-1000,1000]

Optional

Default value = 0

Plus la priorité est basse, plus elle est considérée comme importante. Car il faut lire P0 (priorité = 0) plus prioritaire que P3 (priorité = 3).

Le moteur retirera n’importe quelle quantité d’ordre de priorité importante pour des priorités plus faibles `p0>>>>>>p1`

`priority` est un manière simple et rapide de définir des préférences mais ça n’est pas la plus précise des manières.

## Skills

Array of Strings

Optional

No default value

Les resources possèdent elles aussi un champ `skills` qui indique les attributs ou compétences à leur disposition et qui sont corrélées.

Les `requiredSkills` permettent d’identifier des attributs ou compétences nécessaires à la réalisation des ordres. 

Un exemple classique : les livraisons nécessitant un camion hayon.

Afin de réaliser un ordre, une ressource doit disposer de **la totalité** des `requiredSkills`.

Par exemple :

```json
{
	"id":"ordre-1",
	"requiredSkills":[
		"COMPTER_JUSQUA_10",
		"COMPTER_JUSQUA_30"
	]
}
```

Pour compter jusqu’à 30, il faut savoir compter jusqu’à 10. Inversement on ne sait pas forcément compter jusqu’à 30 (il faut imaginer…).

## VehicleProfile

Enum

- Fly
- Pedestrian
- Bicycle
- Scooter
- Motorbike
- Car
- Truck

Optional

Default value : Fly

`withTraffic` permet de récupérer le trafic prédictif, si la feature est activée (feature payante).

### Fly

Spécificités : `kpmh` , vitesse de vol

```json
{
	"type":"fly",
  "kpmh":30
}
```

## Bicycle / Scooter

Sont interdits d’axe type voie rapide

## Truck

Les camions ont beaucoup plus de paramètres disponibles : 

- `grossWeight` permet d’indiquer la charge utile en kilogrammes
- `avoidTollRoad` permet d’indiquer d’éviter les péages
- `shippedHazardousGoods` permet d’indiquer un transport de matières dangereuses (ADR) parmis cette liste : **`[ explosive, gas, flammable, combustible, organic, poison, radioactive, corrosive, poisonousInhalation, harmfulToWater, other ]`**

## capacities

Key-value map

Optional

No default value

Les `capacities` permettent d’indiquer des contraintes liées aux marchandises. Le format de ce champ est libre pour permettre de déclarer tout type de capacité.

Dans le cas d’une ressource, `capacities` indique la capacité d’emport.

La seule chose requise est de retrouver les mêmes valeurs sur au moins une ressource dans le plan, afin que l’algorithme puisse affecter cet ordre aux ressources éligibles.

Par exemple, pour un coffre multi-température :

- poids à température ambiante : `{ "weight":450.0 }`
- poids dans bac frigorifique : `{ "freezer": 350.0 }`

## departure

Coordinate

Optional

No default value

Indique le point de départ de la ressource donc de la tournée. Habituellement une maison ou un dépôt. Permet d’optimiser le premier trajet.

Si aucun departure n’est fourni, alors l’algorithm considère le temps de travail à partir du premier point.

## arrival

Coordinate

Optional

No default value

Indique le point de fin de tournée/retour de la ressource. Habituellement une maison ou un dépôt. Permet d’optimiser le dernier trajet.

Si aucun arrival n’est fourni, alors l’algorithme considère que  le temps de travail s’arrête à partir du dernier point.

## workingTimeWindow

TimeWindow

Mandatory

No default value (?)

Définit l’amplitude horaire de travail. Souvent utilisé en conjonction avec `maxWorkingDuration`

## maxWorkingDuration

ISO8601 Duration

Optional

Default value = all the working window

Indique la durée maximale travaillée.

<aside>
⚠️ Attention, les pauses sont considérées comme travaillées, si les pauses ne sont pas travaillées, il faut compenser en augmentant la durée de travail.

</aside>

## maxDistanceInKm

Integer

Optional

Default value = Infini

Permet d’indiquer la zone de chalandise à respecter

## breaks

Enum

- **`timeWindowBreak`**
- **`travelDurationSlidingBreak`**
- **`workingDurationSlidingBreak`**

Optional

No default value

Les pauses permettent de décrire des pauses au sens large : déjeuner, légales, temps de formation, temps de manutention…

Les pauses sont considérées comme travaillées.

<aside>
⚠️ Il sera prochainement possible de déclarer des pauses comme étant non travaillées

</aside>

L’algorithm optimise la prise des pauses : 

- S’il est possible de les regrouper ensembles
- S’il est possible de sauter la première
- S’il est possible de finir avant de prendre la dernière

### timeWindowBreak

Permet de déclarer une pause d’une durée `duration`(ISO8601 Duration) à prendre dans un créneau `timeWindow` composée de deux dates (ISO8601 Datetime). C’est une pause standard, la pause déjeuner est un très bon exemple : 

```json
{
	"type":"timeWindowBreak",
	"duration":"PT45M",
	"timeWindow":{
		"begin": "YYYY-MM-DDT12:00:00Z",
		"end": "YYYY-MM-DDT14:00:00Z"
	}
}
```

### travelDurationSlidingBreak

Ce type de pause représente souvent les pauses dites “légales” liés à la conduite

Par exemple : Il faut effectuer une pause de 10 minutes toutes les 4 heures de conduite.

```json
{
  "type": "travelDurationSlidingBreak",
  "minBreakDuration": "PT10M",
  "maxInterBreakDuration": "PT4H"
},
```

### workDurationSlidingBreak

Ce type de pause représente souvent les pauses dites “légales” liés au travail.

Par exemple : Il faut effectuer une pause de 10 minutes toutes les 6 heures de travail.

```json
{
  "type": "travelDurationSlidingBreak",
  "minBreakDuration": "PT10M",
  "maxInterBreakDuration": "PT6H"
},
```

# Properties

Il est possible de fournir des `properties` à plusieurs niveaux : 

- plan
- ressources
- ordres
- stops

Les properties sont dans champs libres de type clé-valeur. Ils vous permettent de passer des paramètres importants à votre compréhension ou intégration.

Elles seront retransmises dans le plan pour tous les niveaux et pour les stops au niveau de la solution.

Nous avons fait le choix de ne pas modéliser de données métier mais nous vous permettons de les utiliser.

# Advanced Modelisations

## SetupDurations

## Costs

## Tags

ressources

```json
"preferredStopTags": [
    "access:parking33",
    "capa:bat22",
    "setup:france"
  ],
  "tags": [
    "trailer"
  ],
```

## AdditionalConstraints

## GlobalConstraints

# Expert Modelisations

- TimeWindows, durations and sleepovers
- Skills as a forcing tool

# Resource state

# Templates xlsx d’exemple

## XLSX Generator ?

# Authentication

> To authorize, use this code:

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -X DELETE \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

