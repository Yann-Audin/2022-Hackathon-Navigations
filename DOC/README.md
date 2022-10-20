# The Greek Anthology's project : our API

The purpose of this notebook is to document the API of the Greek Anthology's project. It is available at http://anthologiagraeca.org/api.
    
## Access the API

We will start by importing different useful libraries and define our first variables for the requests.


```python
import requests
import json

url = 'http://anthologiagraeca.org/api'
parameters = {
    'format':'json'
}

data = requests.get(url, parameters).json()
```

The variable `data` gives us, in json format, the available endpoints


```python
data
```



```
    {'passages': 'https://anthologiagraeca.org/api/passages/?format=json',
     'books': 'https://anthologiagraeca.org/api/books/?format=json',
     'scholia': 'https://anthologiagraeca.org/api/scholia/?format=json',
     'texts': 'https://anthologiagraeca.org/api/texts/?format=json',
     'media': 'https://anthologiagraeca.org/api/media/?format=json',
     'manuscripts': 'https://anthologiagraeca.org/api/manuscripts/?format=json',
     'keywords': 'https://anthologiagraeca.org/api/keywords/?format=json',
     'keyword_categories': 'https://anthologiagraeca.org/api/keyword_categories/?format=json',
     'languages': 'https://anthologiagraeca.org/api/languages/?format=json',
     'descriptions': 'https://anthologiagraeca.org/api/descriptions/?format=json',
     'cities': 'https://anthologiagraeca.org/api/cities/?format=json',
     'authors': 'https://anthologiagraeca.org/api/authors/?format=json',
     'comments': 'https://anthologiagraeca.org/api/comments/?format=json',
     'editions': 'https://anthologiagraeca.org/api/editions/?format=json'}
```


## The endpoint *passages*

The first endpoint (`passages`) is the most important : it contains a list of all the epigrams. Let us have a look at it. 

With the variable "epigrams", we directly make a request to the API (its URL is built thanks to the variables defined above and below).


```python
epigrams = '/passages'
epigrams_res = requests.get(url+epigrams,parameters).json()
```

The result of our request provides us with a lot information related to the passages, as seen below : 


```python
epigrams_res
```



```
    {'count': 4134,
     'next': 'https://anthologiagraeca.org/api/passages/?format=json&page=2',
     'previous': None,
     'results': [{'id': 424,
       'book': {'url': 'https://anthologiagraeca.org/api/books/9/?format=json',
        'number': 1},
       'fragment': 1,
       'sub_fragment': '',
       'urn': 'urn:cts:greekLit:tlg7000.tlg001.ag:1.1',
       'url': 'https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:1.1/?format=json',
       'web_url': '/passages/urn:cts:greekLit:tlg7000.tlg001.ag:1.1/',
       'manuscripts': ['http://digi.ub.uni-heidelberg.de/iiif/2/cpgraec23%3A049.jpg/pct:11.132686084142394,13.202832153269469,60.129449838187696,6.205747605164515/full/0/default.jpg'],
       'texts': [{'language': 'grc',
         'text': '\n                      ἃς οἱ πλάνοι καθεῖλον ἐνθάδ᾽ εἰκόνας\n ἄνακτες ἐστήλωσαν εὐσεβεῖς πάλιν.\n'},
        {'language': 'eng',
         'text': 'Inscribed on the Tabernacle of Saint Sophia\n\nThe images that the hereties took down here our pious sovereigns replaced. '},
        {'language': 'fra',
         'text': "Sur le dais de l'autel de Sainte-Sophie \n\nLes images que des égarés avaient renversées ici, de pieux souverains les ont relevées. "},
        {'language': 'por',
         'text': 'Inscrito no altar de Santa Sofia\n \nDaqui os andarilhos derrubaram as imagens,\nEntão os pios senhores as ergueram novamente.'}],
       'authors': [{'tlg_id': '',
         'names': [{'name': 'Anonimo', 'language': 'ita'},
          {'name': 'Anonyme', 'language': 'fra'}]}],
       'cities': [],
       'keywords': ['https://anthologiagraeca.org/api/keywords/116/?format=json',
        'https://anthologiagraeca.org/api/keywords/200/?format=json'],
       'scholia': [{'book': 1,
         'fragment': 1,
         'sub_fragment': '',
         'number': 1,
         'url': '/passages/urn:cts:greekLit:tlg5011.tlg001.sag:1.1.1/'},
        {'book': 1,
         'fragment': 1,
         'sub_fragment': '',
         'number': 2,
         'url': '/passages/urn:cts:greekLit:tlg5011.tlg001.sag:1.1.2/'}],
       'comments': ['https://anthologiagraeca.org/api/comments/670/?format=json'],
       'external_references': [],
       'internal_references': [],
       'media': []},
       ...

```


As you can see, we have for now 4134 epigrams - the value of `count`.


```python
epigrams_res['count']
```



```python
    4134
```


## Pagination

As you might have noticed, all of the information are not displayed on the block above: the list is pagined and one can navigate the pages using the values of `next` and `previous`.

By default, each pages has: 


```python
len(epigrams_res['results'])
```



```python
    50
```


50 epigrams. You can change this value with the parameter `limit`.


```python
parameters.update({'limit':100})

epigrams_res = requests.get(url+epigrams,parameters).json()

len(epigrams_res['results'])
```



```python
    100
```


This list can be filtered : 
- by book (rendering only the epigrams belonging to one particular book) 
- by author's main name (their English name)
- by keyword id (which can be found on the platform : *e.g.* https://anthologiagraeca.org/keywords/1/ is the URL for the keyword "Elegiac couplet" ; the id is "1". 

> For example: https://anthologiagraeca.org/api/passages/?book__number=5&author__main_name=Meleager&keyword__number=1 will give all the epigrams written by Meleager belonging to book 5 and described with the keyword `1` (Elegiac couplet).


In this next cellule, `alldata` contains all the passages' data for all of our 4134 epigrams! 


```python
url = 'http://anthologiagraeca.org/api/passages'
results = []
pagination = True
while pagination == True :
    alldata = requests.get(url, parameters).json()
    for result in alldata['results'] :
        results.append(result)
    if alldata['next'] is None:
        pagination = False
    else:
        url = alldata['next']
        
alldata
```



```
    {'count': 4134,
     'next': None,
     'previous': 'https://anthologiagraeca.org/api/passages/?format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&format=json&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&limit=100&page=41',
     'results': [{'id': 3695,
       'book': {'url': 'https://anthologiagraeca.org/api/books/16/?format=json',
        'number': 16},
       'fragment': 355,
       'sub_fragment': '',
       'urn': 'urn:cts:greekLit:tlg7000.tlg001.ag:16.355',
       'url': 'https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:16.355/?format=json',
       'web_url': '/passages/urn:cts:greekLit:tlg7000.tlg001.ag:16.355/',
       'manuscripts': [],
       'texts': [{'language': 'grc',
         'text': 'οὔπω σοι μογέοντι Τύχη πόρεν ἄξια νίκης:\n νῖκαι γὰρ τῆς σῆς μείζονες εὐτυχίης.\n\n                   ἀλλὰ μέρει πρώτῳ σταθερῷ καὶ ἀρείονι μίμνοις\n τὴν φθονερὴν τήκων δυσμενέων κραδίην,\n\n                  οἵ, σέθεν εἰσορόωντες ἀεὶ νικῶσαν ἱμάσθλην,\n μέμφονται σφετέρην αἰὲν ἀτασθαλίην.'}],
       'authors': [],
       'cities': [],
       'keywords': [],
       'scholia': [],
       'comments': [],
       'external_references': [],
       'internal_references': [],
       'media': []},
```



```python
len(results)
```



```python
    4134
```


## Data about epigrams

Let us now have a look at the data available for each epigram. Most of these data are present in the list of epigrams (`alldata['results']`), but each epigram has its own endpoint, structured on the basis of its book and its number.

The *Greek Anthology* has 16 books, as you can see here:


```python
number_of_books = requests.get('https://anthologiagraeca.org/api/books/').json()['count']

print(number_of_books)
```
```python
    16
```

An epigram is normally identified by a number (for exemple 1 or 145).

Sometimes there are two or more epigrams for the same number. In these cases letters are used (*e.g.* 132a).

Based on this information the epigram endpoind will be structured as follows:

`/passages/urn:cts:greekLit:tlg7000.tlg001.ag:`+bookNumber`.`+epigramNumber+epigramLetter

> an example from the platform: https://anthologiagraeca.org/passages/urn:cts:greekLit:tlg7000.tlg001.ag:12.132a/

This url is avaiable in the list of epigrams as one can see in the field `url` of each result (let us take the first one here):


```python
epigrams_res['results'][0]['url']
```



```python
    'https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:1.1/?format=json'
```


Let us have a look at the epigram 6.13, which means the epigram number 13 of the book 6:


```python
ep6_13 = requests.get('https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:6.13').json()

ep6_13
```



```
    {'id': 394,
     'book': {'url': 'https://anthologiagraeca.org/api/books/5/', 'number': 6},
     'fragment': 13,
     'sub_fragment': '',
     'urn': 'urn:cts:greekLit:tlg7000.tlg001.ag:6.13',
     'url': 'https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:6.13/',
     'web_url': '/passages/urn:cts:greekLit:tlg7000.tlg001.ag:6.13/',
     'manuscripts': ['http://digi.ub.uni-heidelberg.de/iiif/2/cpgraec23%3A142.jpg/pct:10.614886731391586,73.26114119117034,50.355987055016186,4.664723032069971/full/0/default.jpg',
      'http://digi.ub.uni-heidelberg.de/iiif/2/cpgraec23%3A143.jpg/pct:30.37231055564613,13.349681305818653,51.660362990702126,7.613758509494807/full/0/default.jpg'],
     'texts': [{'language': 'grc',
       'text': 'οἱ τρισσοί τοι ταῦτα τὰ δίκτυα θῆκαν ὅμαιμοι,\nἀγρότα Πάν, ἄλλης ἄλλος ἀπ᾽ ἀγρεσίης:\nὧν ἀπὸ μὲν πτηνῶν Πίγρης τάδε, ταῦτα δὲ Δᾶμις\nτετραπόδων, Κλείτωρ δ᾽ ὁ τρίτος εἰναλίων.\nἀνθ᾽ ὧν τῷ μὲν πέμπε δι᾽ ἠέρος εὔστοχον ἄγρην,\nτῷ δὲ διὰ δρυμῶν, τῷ δὲ δι᾽ ἠϊόνων. '},
      {'language': 'ita',
       'text': 'Tre fratelli ti hanno dedicato queste reti,\nPan cacciatore, ognuna proveniente da una caccia differente.\nPigre queste di uccelli, Damis queste,\ndi bestie feroci, Cleitore, il terzo, di animali marini.\nIn cambio dai una caccia fortunata al primo in aria\nal secondo nei boschi e al terzo sulle rive.'},
      {'language': 'fra',
       'text': "Ces trois frères t'ont dédié ces filets,\nPan chasseur, chacun issu d'une chasse différente.\nPigres celles-ci, d'oiseaux, Damis celle-ci,\nde bêtes féroces, Cléitor, le troisième, d'animaux marins.\nEn échange donne une bonne chasse au premier dans l'air,\nau deuxième dans les bois et au troisième sur les rivages."},
      {'language': 'fra',
       'text': "Les trois frères t'ont consacré, chasseur Pan, ces filets, pris par chacun à son genre de chasse: Pigrès, pour les oiseaux; Damis, pour les quadrupèdes; Cléitor, pour le peuple de la mer. Envoie-leur en échange une bonne chasse à l'un par les airs, au second par les bois, à l'autre par les grèves."},
      {'language': 'eng',
       'text': 'Huntsman Pan, the three brothers dedicated these nets to thee, each from a different chase: Pigres these from fowl, Damis these from beast, and Clitor his from the denizens of the deep. In return for which send them easily caught game, to the first through the air, to the second through the woods, and to the third through the shore-water.'}],
     'authors': [{'tlg_id': 'tlg-1458',
       'names': [{'name': 'Leonidas of Tarentum', 'language': 'eng'},
        {'name': 'Λεωνίδας ὁ Ταραντῖνος', 'language': 'grc'},
        {'name': 'Leonida di Taranto', 'language': 'ita'},
        {'name': 'Λεωνίδας ᾿Αλεξανδρεύς', 'language': 'grc'},
        {'name': "Léonidas d'Alexandrie", 'language': 'fra'},
        {'name': 'Leonidas of Alexandria', 'language': 'eng'}]}],
     'cities': [],
     'keywords': [{'id': 1,
       'names': [{'name': 'distique élégiaque', 'language': 'fra'},
        {'name': 'distico elegiaco', 'language': 'ita'},
        {'name': 'Elegiac couplet', 'language': 'eng'}],
       'category': [{'name': 'Formes métriques', 'language': 'fra'},
        {'name': 'Metric forms', 'language': 'eng'},
        {'name': 'Metra', 'language': 'lat'},
        {'name': 'Forme metriche', 'language': 'ita'},
        {'name': 'Formas métricas', 'language': 'por'}]},
```



```python
ep12_132a = requests.get('https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:12.132a').json()

ep12_132a
```



```
    {'id': 3199,
     'book': {'url': 'https://anthologiagraeca.org/api/books/8/', 'number': 12},
     'fragment': 132,
     'sub_fragment': 'a',
     'urn': 'urn:cts:greekLit:tlg7000.tlg001.ag:12.132a',
     'url': 'https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:12.132a/',
     'web_url': '/passages/urn:cts:greekLit:tlg7000.tlg001.ag:12.132a/',
     'manuscripts': ['http://digi.ub.uni-heidelberg.de/iiif/2/cpgraec23%3A589.jpg/pct:11.78916,41.5646,60.6891,16.87853/full/0/default.jpg'],
     'texts': [{'language': 'grc',
       'text': 'ἆ ψυχὴ βαρύμοχθε, σὺ δ᾽ ἄρτι μὲν ἐκ πυρὸς αἴθῃ,\n ἄρτι δ᾽ ἀναψύχεις, πνεῦμ᾽ ἀναλεξαμένη.\nτί κλαίεις; τὸν ἄτεγκτον ὅτ᾽ ἐν κόλποισιν Ἔρωτα\n ἔτρεφες, οὐκ ᾔδεις ὡς ἐπὶ σοὶ τρέφετο;\n\n                  οὐκ ᾔδεις; νῦν γνῶθι καλῶν ἄλλαγμα τροφείων,\nπῦρ ἅμα καὶ ψυχρὰν δεξαμένη χιόνα.\n αὐτὴ ταῦθ᾽ εἵλου: φέρε τὸν πόνον. ἄξια πάσχεις\nὧν ἔδρας, ὀπτῷ καιομένη μέλιτι.'},
      {'language': 'eng',
       'text': 'O sore-afflicted soul, now thou bumest in the fire and now thou revivest, recovering thy breath. Why dost thou weep? When thou didst nurse merciless Love in thy bosom knewest thou not that he was being nursed for thy bane ? Didst thou not know it ? Now learn to know the pay of thy good nursing, receiving from him fire and cold snow therewith. Thyself thou hast chosen this ; bear the pain. Thou sufferest the due guerdon of what thou hast done, burnt by his boiling honey.'},
      {'language': 'fra',
       'text': 'Oh\xa0! mon âme accablée de souffrances, tantôt c’est le feu qui te brûle, tantôt tu reprends vie en retrouvant le souffle\xa0! Tu pleures\xa0? Lorsque dans ton sein tu nourrissais l’impitoyable Amour, ne savais-tu pas que c’était contre toi que tu le nourrissais\xa0? Tu ne le savais pas\xa0? Vois maintenant le salaire de tes bons soins\xa0: tu reçois tout ensemble feu et neige glacée\xa0! C’est toi, toi qu’il l’a choisi\xa0! Supporte ta douleur\xa0! Juste souffrance de tes actes, la brûlure du miel ardent\xa0!'}],
     'authors': [{'tlg_id': 'tlg-1492',
       'names': [{'name': 'Μελέαγρος', 'language': 'grc'},
        {'name': 'Meleager', 'language': 'eng'},
        {'name': 'Méléagre', 'language': 'fra'}]}],
     'cities': [],
     'keywords': [{'id': 1,
       'names': [{'name': 'distique élégiaque', 'language': 'fra'},
        {'name': 'distico elegiaco', 'language': 'ita'},
        {'name': 'Elegiac couplet', 'language': 'eng'}],
       'category': [{'name': 'Formes métriques', 'language': 'fra'},
        {'name': 'Metric forms', 'language': 'eng'},
        {'name': 'Metra', 'language': 'lat'},
        {'name': 'Forme metriche', 'language': 'ita'},
        {'name': 'Formas métricas', 'language': 'por'}]},
      {'id': 3,
       'names': [{'name': 'erotic', 'language': 'eng'},
        {'name': 'érotic', 'language': 'fra'},
        {'name': 'erotico', 'language': 'ita'}],
       'category': [{'name': 'Genres', 'language': 'fra'},
        {'name': 'Genres', 'language': 'eng'},
        {'name': 'Genera', 'language': 'lat'},
        {'name': 'Generi', 'language': 'ita'},
        {'name': 'Gêneros', 'language': 'por'}]},
      {'id': 4,
       'names': [{'name': 'époque hellénistique', 'language': 'fra'},
        {'name': 'epoca ellenistica', 'language': 'ita'},
        {'name': 'hellenistic period', 'language': 'eng'}],
       'category': [{'name': 'Époques', 'language': 'fra'},
        {'name': 'Periods', 'language': 'eng'},
        {'name': 'Tempora', 'language': 'lat'},
        {'name': 'Periodi', 'language': 'ita'},
        {'name': 'Épocas', 'language': 'por'}]},
      {'id': 181,
       'names': [{'name': 'Couronne de Méléagre', 'language': 'fra'}],
       'category': [{'name': 'Collections', 'language': 'fra'},
        {'name': 'Collections', 'language': 'eng'},
        {'name': 'Collecteana', 'language': 'lat'},
        {'name': 'Collezioni', 'language': 'ita'},
        {'name': 'Coleções', 'language': 'por'}]}],
     'scholia': [],
     'comments': [],
     'external_references': [],
     'internal_references': [],
     'media': []}
```


The epigram's number is in the key `fragment` and the letter (when it has one) in the key `sub_fragment`

### Images of the manuscript (Codex Palatinus 23)

For each epigram, the corresponding iiif images of the manuscript can be found under the key `manuscript` (a high quality digitization of the *codex palatinus 23* is available on [the website of the Palatinate Library of Heidelberg](https://digi.ub.uni-heidelberg.de/diglit/cpgraec23/0079/image,info)). 

For more information about the manuscript and its images, cf. the section "Manuscript Annotation API" in this document. 


```python
ep6_13['manuscripts']
```



```python
    ['http://digi.ub.uni-heidelberg.de/iiif/2/cpgraec23%3A142.jpg/pct:10.614886731391586,73.26114119117034,50.355987055016186,4.664723032069971/full/0/default.jpg',
     'http://digi.ub.uni-heidelberg.de/iiif/2/cpgraec23%3A143.jpg/pct:30.37231055564613,13.349681305818653,51.660362990702126,7.613758509494807/full/0/default.jpg']
```


Two images are associated with epigram 6.13 since it spans two different pages. 

### Texts

Each epigram has a list of texts wich are associated to it. All the epigrams in our database should have at least the greek text. An epigram can have more than one greek editions of the text and a set of translations in different languages :


```python
ep6_13['texts']
```



```
    [{'language': 'grc',
      'text': 'οἱ τρισσοί τοι ταῦτα τὰ δίκτυα θῆκαν ὅμαιμοι,\nἀγρότα Πάν, ἄλλης ἄλλος ἀπ᾽ ἀγρεσίης:\nὧν ἀπὸ μὲν πτηνῶν Πίγρης τάδε, ταῦτα δὲ Δᾶμις\nτετραπόδων, Κλείτωρ δ᾽ ὁ τρίτος εἰναλίων.\nἀνθ᾽ ὧν τῷ μὲν πέμπε δι᾽ ἠέρος εὔστοχον ἄγρην,\nτῷ δὲ διὰ δρυμῶν, τῷ δὲ δι᾽ ἠϊόνων. '},
     {'language': 'ita',
      'text': 'Tre fratelli ti hanno dedicato queste reti,\nPan cacciatore, ognuna proveniente da una caccia differente.\nPigre queste di uccelli, Damis queste,\ndi bestie feroci, Cleitore, il terzo, di animali marini.\nIn cambio dai una caccia fortunata al primo in aria\nal secondo nei boschi e al terzo sulle rive.'},
     {'language': 'fra',
      'text': "Ces trois frères t'ont dédié ces filets,\nPan chasseur, chacun issu d'une chasse différente.\nPigres celles-ci, d'oiseaux, Damis celle-ci,\nde bêtes féroces, Cléitor, le troisième, d'animaux marins.\nEn échange donne une bonne chasse au premier dans l'air,\nau deuxième dans les bois et au troisième sur les rivages."},
     {'language': 'fra',
      'text': "Les trois frères t'ont consacré, chasseur Pan, ces filets, pris par chacun à son genre de chasse: Pigrès, pour les oiseaux; Damis, pour les quadrupèdes; Cléitor, pour le peuple de la mer. Envoie-leur en échange une bonne chasse à l'un par les airs, au second par les bois, à l'autre par les grèves."},
     {'language': 'eng',
      'text': 'Huntsman Pan, the three brothers dedicated these nets to thee, each from a different chase: Pigres these from fowl, Damis these from beast, and Clitor his from the denizens of the deep. In return for which send them easily caught game, to the first through the air, to the second through the woods, and to the third through the shore-water.'}]
```


### Authors


An epigram is almost always associated to one or more authors (since the attributions are often uncertain):


```python
ep6_13['authors']
```



```
    [{'tlg_id': 'tlg-1458',
      'names': [{'name': 'Leonidas of Tarentum', 'language': 'eng'},
       {'name': 'Λεωνίδας ὁ Ταραντῖνος', 'language': 'grc'},
       {'name': 'Leonida di Taranto', 'language': 'ita'},
       {'name': 'Λεωνίδας ᾿Αλεξανδρεύς', 'language': 'grc'},
       {'name': "Léonidas d'Alexandrie", 'language': 'fra'},
       {'name': 'Leonidas of Alexandria', 'language': 'eng'}]}]
```


### Keywords

Each epigram can be associated with keywords.

One can have more information about a keyword on its own endpoint, structured as follow : 

`https://anthologiagraeca.org/api/keywords/`+keyword_id 

(the keyword id can be found here at the end of its URL on the platform, *e.g.* https://anthologiagraeca.org/keywords/1/ is the URL for the keyword "Elegiac couplet" ; the id is "1").


```python
ep6_13['keywords']
```



```
    [{'id': 1,
      'names': [{'name': 'distique élégiaque', 'language': 'fra'},
       {'name': 'distico elegiaco', 'language': 'ita'},
       {'name': 'Elegiac couplet', 'language': 'eng'}],
      'category': [{'name': 'Formes métriques', 'language': 'fra'},
       {'name': 'Metric forms', 'language': 'eng'},
       {'name': 'Metra', 'language': 'lat'},
       {'name': 'Forme metriche', 'language': 'ita'},
       {'name': 'Formas métricas', 'language': 'por'}]},
     {'id': 4,
      'names': [{'name': 'époque hellénistique', 'language': 'fra'},
       {'name': 'epoca ellenistica', 'language': 'ita'},
       {'name': 'hellenistic period', 'language': 'eng'}],
      'category': [{'name': 'Époques', 'language': 'fra'},
       {'name': 'Periods', 'language': 'eng'},
       {'name': 'Tempora', 'language': 'lat'},
       {'name': 'Periodi', 'language': 'ita'},
       {'name': 'Épocas', 'language': 'por'}]},
     {'id': 73,
      'names': [{'name': 'Validé par William', 'language': 'fra'}],
      'category': [{'name': 'Validation', 'language': 'fra'},
       {'name': 'Validation', 'language': 'eng'},
       {'name': 'Contralectus', 'language': 'lat'},
       {'name': 'Validazione', 'language': 'ita'},
       {'name': 'Validação', 'language': 'por'}]},
     {'id': 186,
      'names': [{'name': 'dedicatory', 'language': 'eng'},
       {'name': 'votif (anathématique)', 'language': 'fra'}],
      'category': [{'name': 'Genres', 'language': 'fra'},
       {'name': 'Genres', 'language': 'eng'},
       {'name': 'Genera', 'language': 'lat'},
       {'name': 'Generi', 'language': 'ita'},
       {'name': 'Gêneros', 'language': 'por'}]},
     {'id': 234,
      'names': [{'name': 'Damis', 'language': 'eng'}],
      'category': [{'name': 'Personnes citées', 'language': 'fra'},
       {'name': 'Quoted persons', 'language': 'eng'},
       {'name': 'Homines memorati', 'language': 'lat'},
       {'name': 'Persone citate', 'language': 'ita'},
       {'name': 'Pessoas citadas', 'language': 'por'}]},
     {'id': 270,
      'names': [{'name': 'Chasse', 'language': 'fra'}],
      'category': [{'name': 'Motifs', 'language': 'fra'},
       {'name': 'Motifs', 'language': 'eng'},
       {'name': 'Themata', 'language': 'lat'},
       {'name': 'Motivi', 'language': 'ita'},
       {'name': 'Motivos', 'language': 'por'}]},
     {'id': 492,
      'names': [{'name': 'Pan', 'language': 'eng'}],
      'category': [{'name': 'Divinités', 'language': 'fra'},
       {'name': 'Deities', 'language': 'eng'},
       {'name': 'Divinitates', 'language': 'lat'},
       {'name': 'Divinità', 'language': 'ita'},
       {'name': 'Divindades', 'language': 'por'}]},
     {'id': 574,
      'names': [{'name': 'Πίγρης', 'language': 'grc'},
       {'name': 'Pigrès', 'language': 'fra'}],
      'category': [{'name': 'Personnes citées', 'language': 'fra'},
       {'name': 'Quoted persons', 'language': 'eng'},
       {'name': 'Homines memorati', 'language': 'lat'},
       {'name': 'Persone citate', 'language': 'ita'},
       {'name': 'Pessoas citadas', 'language': 'por'}]},
     {'id': 575,
      'names': [{'name': 'Κλείτωρ', 'language': 'grc'},
       {'name': 'Cléitor', 'language': 'fra'}],
      'category': [{'name': 'Personnes citées', 'language': 'fra'},
       {'name': 'Quoted persons', 'language': 'eng'},
       {'name': 'Homines memorati', 'language': 'lat'},
       {'name': 'Persone citate', 'language': 'ita'},
       {'name': 'Pessoas citadas', 'language': 'por'}]}]
```



```python
keyword_1 = requests.get("https://anthologiagraeca.org/api/keywords/1/").json()
```


```python
keyword_1
```



```
    {'id': 1,
     'url': 'https://anthologiagraeca.org/api/keywords/1/',
     'category': {'id': 1,
      'url': 'https://anthologiagraeca.org/api/keyword_categories/1/',
      'names': [{'name': 'Formes métriques', 'language': 'fra'},
       {'name': 'Metric forms', 'language': 'eng'},
       {'name': 'Metra', 'language': 'lat'},
       {'name': 'Forme metriche', 'language': 'ita'},
       {'name': 'Formas métricas', 'language': 'por'}]},
     'names': [{'name': 'distique élégiaque', 'language': 'fra'},
      {'name': 'distico elegiaco', 'language': 'ita'},
      {'name': 'Elegiac couplet', 'language': 'eng'}],
     'alternative_urns': [{'urn': 'https://www.wikidata.org/wiki/Q2082412'}]}
```


keywords are organized in `categories` and each keyword **must** belong to a category.

The list of the categories is here: https://anthologiagraeca.org/api/keyword_categories/

### Cities (places)

An epigram can be associated with some places (cities or countries or whatever can have geocoordinates). Epigram 6.13 has no cities associated. Let us look to another epigram:


```python
ep7_81 = requests.get('https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:7.81').json()
ep7_81['cities']
```



```
    [{'id': 45, 'names': [{'name': 'Lindus', 'language': 'eng'}]},
     {'id': 61,
      'names': [{'name': 'Atenas', 'language': 'por'},
       {'name': 'Classical Athens', 'language': 'eng'},
       {'name': 'Antigua Atenas', 'language': 'spa'},
       {'name': 'Athènes', 'language': 'fra'},
       {'name': 'Atene', 'language': 'ita'},
       {'name': 'Athenae antiquae', 'language': 'lat'}]},
     {'id': 73,
      'names': [{'name': 'Sparta', 'language': 'eng'},
       {'name': 'Sparte', 'language': 'fra'},
       {'name': 'Esparta', 'language': 'spa'},
       {'name': 'Sparta', 'language': 'ita'},
       {'name': 'Sparta', 'language': 'lat'},
       {'name': 'Esparta', 'language': 'por'}]},
     {'id': 78,
      'names': [{'name': 'Antigua Corinto', 'language': 'spa'},
       {'name': 'Ancient Corinth', 'language': 'eng'},
       {'name': 'Corinthe antique', 'language': 'fra'},
       {'name': 'Corinto', 'language': 'ita'}]},
     {'id': 89,
      'names': [{'name': 'Mytilene', 'language': 'eng'},
       {'name': 'Mytilène', 'language': 'fra'},
       {'name': 'Mitilene', 'language': 'spa'},
       {'name': 'Mitilene', 'language': 'ita'},
       {'name': 'Mytilene', 'language': 'lat'}]},
     {'id': 128,
      'names': [{'name': 'Milet', 'language': 'fra'},
       {'name': 'Mileto', 'language': 'spa'},
       {'name': 'Mileto', 'language': 'por'},
       {'name': 'Mileto', 'language': 'ita'},
       {'name': 'Miletus', 'language': 'eng'},
       {'name': 'Miletus', 'language': 'lat'},
       {'name': 'Mileto', 'language': 'spa'},
       {'name': 'Mileto', 'language': 'por'},
       {'name': 'Milet', 'language': 'fra'},
       {'name': 'Mileto', 'language': 'ita'},
       {'name': 'Miletus', 'language': 'eng'},
       {'name': 'Miletus', 'language': 'lat'}]},
     {'id': 129,
      'names': [{'name': 'Priene', 'language': 'eng'},
       {'name': 'Priene', 'language': 'spa'},
       {'name': 'Priène', 'language': 'fra'},
       {'name': 'Priene', 'language': 'ita'},
       {'name': 'Priene', 'language': 'por'},
       {'name': 'Priene', 'language': 'lat'}]}]
```


And let us have a look to one of these cities:


```python
Mytilene = requests.get('https://anthologiagraeca.org/api/cities/89').json()
Mytilene
```



```
    {'url': 'https://anthologiagraeca.org/api/cities/89/',
     'names': [{'name': 'Mytilene', 'language': 'eng'},
      {'name': 'Mytilène', 'language': 'fra'},
      {'name': 'Mitilene', 'language': 'spa'},
      {'name': 'Mitilene', 'language': 'ita'},
      {'name': 'Mytilene', 'language': 'lat'}],
     'alternative_urns': ['https://www.wikidata.org/wiki/Q42295059',
      'https://pleiades.stoa.org/places/550763'],
     'unique_id': 95,
     'longitude': None,
     'latitude': None,
     'descriptions': [],
     'created_at': '2021-09-09T19:09:12.954728Z',
     'updated_at': '2021-09-09T19:09:12.954734Z'}
```


### Scholia

On the *Codex Palatinus 23*, *scholia* are often associated to epigrams. Those *scholia* are also rendered on the platform and hence on the API. 


```python
ep6_13['scholia']
```



```
    [{'book': 6,
      'fragment': 13,
      'sub_fragment': '',
      'number': 1,
      'url': '/passages/urn:cts:greekLit:tlg5011.tlg001.sag:6.13.1/'},
     {'book': 6,
      'fragment': 13,
      'sub_fragment': '',
      'number': 2,
      'url': '/passages/urn:cts:greekLit:tlg5011.tlg001.sag:6.13.2/'}]
```


Each *scholium* is identified by the epigram to which it is a *scholium* plus a number. For exemple the second *scholium* of the epigram 6.13 will be named 6.13.2. 

The urn of a scholium will be structured as follows:

`/passages/urn:cts:greekLit:tlg5011.tlg001.sag:`+bookNumber.+epigramNumber+epigramLetter`.`+scholiumNumber

> For example: `/passages/urn:cts:greekLit:tlg5011.tlg001.sag:6.13.2/`

A scholium contains very similar information as an epigram: texts in different languages, the picture of the manuscript (in iiif), cities, keywords, commentaries.

**Attention** the texts opening a book have been identified as scholia to the epigram 0 of the book. For example 5.0.1 or 5.0.2 or 12.0.1


```python
sc5_0_1 = requests.get('https://anthologiagraeca.org/api/scholia/urn:cts:greekLit:tlg5011.tlg001.sag:5.0.1/').json()
print(sc5_0_1)
sc5_0_2 = requests.get('https://anthologiagraeca.org/api/scholia/urn:cts:greekLit:tlg5011.tlg001.sag:5.0.2/').json()
print(sc5_0_2)
sc12_0_2 = requests.get('https://anthologiagraeca.org/api/scholia/urn:cts:greekLit:tlg5011.tlg001.sag:12.0.2/').json()
print(sc12_0_2)
```
```
    {'id': 1999, 'url': 'https://anthologiagraeca.org/api/scholia/urn:cts:greekLit:tlg5011.tlg001.sag:5.0.1/', 'urn': 'urn:cts:greekLit:tlg5011.tlg001.sag:5.0.1', 'web_url': '/passages/urn:cts:greekLit:tlg5011.tlg001.sag:5.0.1/', 'manuscripts': ['http://digi.ub.uni-heidelberg.de/iiif/2/cpgraec23%3A087.jpg/pct:9.335021316210499,67.9883147400695,17.828050776662216,8.264462809917363/full/0/default.jpg'], 'texts': [{'language': 'grc', 'text': 'ἀρχή τῶν ἐρωτικὼν ἐπιγραμμάτων'}, {'language': 'por', 'text': 'Início dos epigramas eróticos.'}, {'language': 'fra', 'text': 'Le début des épigrammes érotiques.'}], 'cities': [], 'keywords': [], 'passage': {'book': 5, 'fragment': 0, 'sub_fragment': '', 'url': '/passages/urn:cts:greekLit:tlg7000.tlg001.ag:5.0/'}, 'comments': [{'description': ['Lemmatiste: L.'], 'language': ['fra']}], 'media': []}
    {'id': 2000, 'url': 'https://anthologiagraeca.org/api/scholia/urn:cts:greekLit:tlg5011.tlg001.sag:5.0.2/', 'urn': 'urn:cts:greekLit:tlg5011.tlg001.sag:5.0.2', 'web_url': '/passages/urn:cts:greekLit:tlg5011.tlg001.sag:5.0.2/', 'manuscripts': [' http://digi.ub.uni-heidelberg.de/iiif/2/cpgraec23%3A088.jpg/pct:9.57839977186465,5.684292685810079,77.95889161681437,5.6234937070427575/full/0/default.jpg'], 'texts': [{'language': 'grc', 'text': 'ἀρχή τῶν ἐρωτικῶν έπιγραμμάτων - διαφόρων ποιητῶν'}, {'language': 'por', 'text': 'Início dos epigramas eróticos - de diferentes poetas.'}, {'language': 'fra', 'text': 'Le début des épigrammes érotiques - de différentes auteurs.'}], 'cities': [], 'keywords': [], 'passage': {'book': 5, 'fragment': 0, 'sub_fragment': '', 'url': '/passages/urn:cts:greekLit:tlg7000.tlg001.ag:5.0/'}, 'comments': [], 'media': []}
    {'id': 816, 'url': 'https://anthologiagraeca.org/api/scholia/urn:cts:greekLit:tlg5011.tlg001.sag:12.0.2/', 'urn': 'urn:cts:greekLit:tlg5011.tlg001.sag:12.0.2', 'web_url': '/passages/urn:cts:greekLit:tlg5011.tlg001.sag:12.0.2/', 'manuscripts': ['http://digi.ub.uni-heidelberg.de/iiif/2/cpgraec23%3A569.jpg/pct:12.639237143536691,11.953945008805782,63.97128581066486,12.422816818582767/full/0/default.jpg'], 'texts': [{'language': 'grc', 'text': 'καὶ τίς ἂν εἴην εἰ πάντων σοι τῶν εἰρημένων τὴν γνῶσιν ἐκθέμενος τῶν Στράτωνος τῶν Σαρδιανοῦ Παιδικὴν Μοῦσαν ἐπεκρυψάμην, ἣν αὐτὸς παίζων πρὸς τοὺς πλησίον ἐπεδείκνυτο, τέρψιν οἰκείαν τὴν ἀπαγγελίαν τῶν ἐπιγραμμάτων, οὐ τὸν νοῦν, ποιούμενος. ἔχου τοίνυν τῶν ἑξῆς: ἐν χορείαις γὰρ ἥ γε σώφρων, κατὰ τὸν τραγικόν, οὐ διαφθαρήσεται.'}, {'language': 'grc', 'text': 'καὶ τίς ἂν εἴην εἰ πάντων σοι τῶν εἰρημένων τὴν γνῶσιν ἐκθέμενος τὴν Στράτωνος τοῦ Σαρδιανοῦ Παιδικὴν Μοῦσαν ἐπεκρυψάμην, ἣν αὐτὸς παίζων πρὸς τοὺς πλησίον ἐπεδείκνυτο, τέρψιν οἰκείαν τὴν ἀπαγγελίαν τῶν ἐπιγραμμάτων, οὐ τὸν νοῦν, ποιούμενος. ἔχου τοίνυν τῶν ἑξῆς: ἐν χορείαις γὰρ ἥ γε σώφρων, κατὰ τὸν τραγικόν, οὐ διαφθαρήσεται.'}, {'language': 'eng', 'text': 'And what kind of man should I be, reader, if after setting forth all that precedes for thee to study, I were to conceal the Puerile Muse of Strato of Barclis, which he used to recite to those about him in sport, taking personal delight in the diction of the epigrams, not in their meaning. Apply thyself then to what follows, for “in dances,” as the tragic poet says, "a chaste woman will not be corrupted.”'}, {'language': 'fra', 'text': "Et qui serais-je si après t'avoir donné la connaissance de toutes les choses dites, je te cachais la Muse garçonnière de Straton de Sardes, que lui-même en jouant récitait à ceux qui l'entouraient, en se faisant une joie personnelles de la récitation des épigrammes et non de leur sens. Voilà pour toi ce qui suit: dans les danses en effet, comme le dit le poète tragique, une femme chaste ne peut pas être corrompue."}], 'cities': [], 'keywords': [], 'passage': {'book': 12, 'fragment': 0, 'sub_fragment': '', 'url': '/passages/urn:cts:greekLit:tlg7000.tlg001.ag:12.0/'}, 'comments': [], 'media': []}
```

### Comments

An epigram can have comments, a sort of footnotes about the epigram as a whole of some part of it. These comments can be multilingual and they sometimes contain some markdown markup:


```python
ep6_13['comments']
```



```
    [{'description': ['# Fresque de Pompéi\n\n Cette épigramme est représentée, selon Gutzwiller, dans une fresque de Pompéi'],
      'language': ['fra']},
     {'description': ["# Imitations de 6.13\n\n C'est vraisemblablement cette pièce de Léonidas qui a inspiré les épigr. 11-16 et 179-187, entre autres 14 (d'Antipater) et 16 (d'Archias), qui n'en sont que des imitations assez serviles. Elle était, en tous cas, une des plus classiques de toute la série; car c'est celle-là qu'on avait gravée sur le mur d'une maison de Pompéi pour servir de légende à une scène où était peinte cette offrande de trois chasseurs; mais il ne subsiste, des trois distiques que comportait l'inscription, que cinq lettres disséminées et les six premières du v. 6; et il a fallu toute la perspicacité de Dilthey pour y reconnaître des fragments de notre épigramme. \n-P. Waltz "],
      'language': ['fra']}]
```


### External references

An epigram can be linked to some external source. The idea of the project is to develop an edition which can take into account some "weak links" between the anthological material and some other cultural artefacts. This links are subjective and they do not want to be considered as "scientific". They can be link to a pop song, for example.

They are just hyperlinks. For example:


```python
ep7_710 = requests.get('https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:7.710').json()
ep7_710['external_references']
```



```
    [{'title': 'Jacques Lazure, Le Blaffart (2018)',
      'url': 'https://opuscules.ca/article-alire?article=202248'},
     {'title': 'Charles Baudelaire, Remords Posthumes (1857)',
      'url': 'https://poesie.webnet.fr/lesgrandsclassiques/Poemes/charles_baudelaire/remords_posthume'},
     {'title': 'Mylène Farmer, Regrets (1991)',
      'url': 'https://www.youtube.com/watch?v=ph6piqBnkgU'}]
```


### Internal references

Internal references are links between epigrams of the anthology. The relationship is symmetrical. 

Many types of links are possible. An epigram can be the variation of another (and there are different kind of variation) or just have a vague link. 

This is still work in progress ; the type of internal reference will most probably always be "default".


```python
ep6_13['internal_references']
```



```
    [{'to_passage': 'https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:6.11/',
      'reference_type': 'Default'},
     {'to_passage': 'https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:6.14/',
      'reference_type': 'Default'},
     {'to_passage': 'https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:6.179/',
      'reference_type': 'Default'},
     {'to_passage': 'https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:6.186/',
      'reference_type': 'Default'},
     {'to_passage': 'https://anthologiagraeca.org/api/passages/urn:cts:greekLit:tlg7000.tlg001.ag:6.187/',
      'reference_type': 'Default'}]
```


## Wikidata and other identifiers

Almost all the objects in our data are aligned with a wikidata id. It is the case for `authors`, `keywords` and `cities`. The wikidata id can be found in the field `alternative_urns`.

Let us look for an author: 


```python
meleager = requests.get('https://anthologiagraeca.org/api/authors/1').json()
```


```python
meleager
```



```
    {'id': 1,
     'url': 'https://anthologiagraeca.org/api/authors/1/',
     'names': [{'name': 'Callimachus', 'language': 'eng'},
      {'name': 'Καλλίμαχος', 'language': 'grc'},
      {'name': 'Callimaque', 'language': 'fra'}],
     'alternative_urns': ['https://www.wikidata.org/wiki/Q192417',
      'http://data.perseus.org/catalog/urn:cts:greekLit:tlg0533'],
     'city_born': {'url': 'https://anthologiagraeca.org/api/cities/1/',
      'names': [{'name': 'Cyrène', 'language': 'fra'},
       {'name': 'Cirene', 'language': 'spa'},
       {'name': 'Cirene', 'language': 'por'},
       {'name': 'Cyrene', 'language': 'eng'},
       {'name': 'Cyrene', 'language': 'lat'},
       {'name': 'Cirene', 'language': 'ita'}],
      'alternative_urns': ['https://anthologia.ecrituresnumeriques.ca/api/v1/cities/3',
       'https://www.wikidata.org/wiki/Q44112',
       'https://pleiades.stoa.org/places/373778'],
      'unique_id': 3,
      'longitude': '32.80799',
      'latitude': '21.86616',
      'descriptions': [],
      'created_at': '2021-04-08T21:27:25.373000Z',
      'updated_at': '2021-04-08T21:27:25.373000Z'},
     'city_died': {'url': 'https://anthologiagraeca.org/api/cities/2/',
      'names': [{'name': 'Alexandrie', 'language': 'fra'},
       {'name': 'Alexandria', 'language': 'eng'},
       {'name': 'Ἀλεξάνδρεια', 'language': 'grc'}],
      'alternative_urns': ['https://anthologia.ecrituresnumeriques.ca/api/v1/cities/4',
       'https://www.wikidata.org/wiki/Q87',
       'https://pleiades.stoa.org/places/727070'],
      'unique_id': 4,
      'longitude': '31.20010',
      'latitude': '29.91857',
      'descriptions': [],
      'created_at': '2021-04-08T21:27:25.384000Z',
      'updated_at': '2021-04-08T21:27:25.384000Z'},
     'born_range_year_date': None,
     'died_range_year_date': None,
     'unique_id': 1,
     'created_at': '2021-04-08T21:27:25.392000Z',
     'updated_at': '2021-04-08T21:27:25.392000Z',
     'validation': 0,
     'tlg_id': 'tlg-0533',
     'main_name': 'Callimachus',
     'descriptions': [],
     'images': []}
```



```python
meleager['alternative_urns']
```



```
    ['https://www.wikidata.org/wiki/Q192417',
     'http://data.perseus.org/catalog/urn:cts:greekLit:tlg0533']
```


Other identifiers can be found in alternative_urns: for exemple the TLG for authors (look above) and the pleiades id for places:


```python
Mytilene['alternative_urns']
```



```
    ['https://www.wikidata.org/wiki/Q42295059',
     'https://pleiades.stoa.org/places/550763']
```


## Manuscript Annotation API

We will shortly describe the API developped by the Palatine Library of Heidelberg regarding the annotation of the manuscript. 

All the annotations can be retrieved as follows:

The manuscript has 649 pages. If one wants all the annotation `a` should be 0 and `b` 648. 

**It is only the Codex Palatinus 23 and not the Sup Graec 384!**

Indeed, the Palatine Library of Heidelberg contains the [first 13 books of the Palatine Anthology](https://digi.ub.uni-heidelberg.de/diglit/cpgraec23/0079/image,info) (from pages 49 to 614). Books 14 et 15 (pages 615 to 709) are held at the Bibliothèque Nationale de France under the name [*Parisinus Supplementum Graecum 384*](https://gallica.bnf.fr/ark:/12148/btv1b8470199g). 

Let us look just at some annotations:


```python
def getAnnotations(a,b):
    result = {}
   
    for page in range(a,b):
       
        page = "{:04d}".format(page)
        url = 'https://anno.ub.uni-heidelberg.de/anno/anno?$regex=true&$target=https://digi.ub.uni-heidelberg.de/diglit/cpgraec23/'+page
        r = requests.get(url)
        
        data = r.json()
        result[page] = []
        
        if data['total'] > 0:
            for item in data['first']['items']:
                result[page].append(item)
                

    return result
```


```python
getAnnotations(590,599)
```



```
    {'0590': [{'id': 'https://anno.ub.uni-heidelberg.de/anno/anno/I9jkuBuzRb6lJ193QTK80A',
       'type': 'Annotation',
       'title': 'Epigram 11.401 (part 2/2)',
       'collection': 'diglit',
       'body': [{'purpose': 'classifying',
         'source': 'https://d-nb.info/gnd/1231592788',
         'value': 'Pseudo-Lucianus Samosatensis',
         'label': 'Pseudo-Lucianus Samosatensis'},
        {'purpose': 'linking',
         'predicate': 'http://terminology.lido-schema.org/lido00263',
         'source': 'https://anthologiagraeca.org/passages/urn:cts:greekLit:tlg7000.tlg001.ag:11.401/',
         'value': 'Anthologia Graeca Project'},
        {'purpose': 'linking',
         'predicate': 'http://terminology.lido-schema.org/lido00263',
         'source': 'http://data.perseus.org/citations/urn:cts:greekLit:tlg7000.tlg001.perseus-grc4:11.401',
         'value': 'Perseus'}],
       'target': {'scope': 'https://digi.ub.uni-heidelberg.de/diglit/cpgraec23/0590',
        'source': 'https://digi.ub.uni-heidelberg.de/diglit/cpgraec23/0590/_image',
        'selector': {'type': 'SvgSelector',
         'value': '<?xml version="1.0" encoding="UTF-8" ?>\n<svg xmlns="http://www.w3.org/2000/svg" version="1.1"\n  width="3242">\n  <rect x="894.6341353255123" y="583.9104347553645" width="1909.1860465116272" height="361.77821894498004" />\n</svg>'}},
       'rights': 'https://creativecommons.org/licenses/by/4.0/',
       'creator': {'displayName': 'Maxime Guénette',
        'id': 'p1141678@umontreal.ca'},
       'hasVersion': [{'versionOf': 'https://anno.ub.uni-heidelberg.de/anno/anno/I9jkuBuzRb6lJ193QTK80A',
       ...
]}]}
```