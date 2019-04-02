# Install ElasticSearch (ES)
I had installed ElasticSearch 6.6.2
## Run
```
.\elasticsearch.bat
```

# Install an ES plugin for Polish
```
.\elasticsearch-plugin install pl.allegro.tech.elasticsearch.plugin:elasticsearch-analysis-morfologik:6.6.2
```
# Define an ES analyzer for Polish texts containing

```
PUT /lab2_index
{
    "settings": {
        "analysis": {
            "analyzer": {
                "lab2_analyzer": {
                    "type": "custom",
                    "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                        "synonym_filter",
                        "morfologik_stem"
                    ]
                }
            },
            "filter": {
                "synonym_filter": {
                    "type": "synonym",
                    "synonyms": [
                        "kpk=>kodeks postępowania karnego",
                        "kpc=>kodeks postępowania cywilnego",
                        "kk=>kodeks karny",
                        "kc=>kodeks cywilny"
                    ]
                }
            }
        }
    },
    "mappings": {
        "act": {
            "properties": {
                "content": {
                    "type": "text",
                    "analyzer": "lab2_analyzer"
                }
            }
        }
    }
}
'
```

# Define an ES index for storing the contents of the legislative acts.

# Load the data to the ES index.

```
import os
import requests


def filesNames():
    path = './ustawy'
    absolute_path = os.getcwd() + "\\ustawy\\"
    return [(absolute_path + filename, filename) for filename in os.listdir(path)]

def getFileTextRaw(filename):
    with open(filename, 'r', encoding="utf8") as content_file:
        return content_file.read()

def sendData(filename, content):
    name = filename.replace('.txt', '')
    url = 'http://localhost:9200/lab2_index/act/{}'.format(name)
    headers = {'Content-Type': 'application/json'}
    data = {'content': content}
    requests.put(url=url, json=data, headers=headers)
    
for (path, filename) in filesNames():
    content = getFileTextRaw(path)
    sendData(filename, content)
```

# Determine the number of legislative acts containing the word ustawa (in any form).

```
GET /lab2_index/act/_search
{
  "query": {
    "match": {
      "content": "ustawa"
    }
  }
}
```

Result: 1179

# Determine the number of legislative acts containing the words kodeks postępowania cywilnego in the specified order, but in an any inflection form.

```
GET /lab2_index/act/_search
{
  "query": {
    "match_phrase": {
      "content": {
        "query": "kodeks postępowania cywilnego",
        "slop": 0
      }
    }
  }
}
```

Result: 100

# Determine the number of legislative acts containing the words wchodzi w życie (in any form) allowing for up to 2 additional words in the searched phrase.

```
GET /lab2_index/act/_search
{
  "query": {
    "match_phrase": {
      "content": {
        "query": "wchodzi w życie",
        "slop": 2
      }
    }
  }
}
```

Result: 1175

# Determine the 10 documents that are the most relevant for the phrase konstytucja.

```
GET /lab2_index/act/_search
{
  "query": {
    "match": {
      "content": "konstytucja"
    }
  },
  "size": 10
}
```

Result:
```
[{"_id":"2000_443","_score":7.366205},
{"_id":"1997_604","_score":7.10154},
{"_id":"1996_350","_score":7.1004834},
{"_id":"1997_642","_score":6.9129972},
{"_id":"1997_629","_score":6.5681167},
{"_id":"1999_688","_score":6.2509775},
{"_id":"1996_199","_score":5.62988},
{"_id":"1997_681","_score":5.360203},
{"_id":"2001_23","_score":5.318322},
{"_id":"2001_247","_score":5.0927725}]
```


# Print the excerpts containing the word konstytucja (up to three excerpts per document) from the previous task.

```
{
  "query": {
    "match": {
      "content": "konstytucja"
    }
  },
  "size": 10,
  "highlight": {
    "fields": {
      "content": {
      	"number_of_fragments": 3
      }
    }
  }
}
```

Result:
```
[{"_id":"2000_443","_score":7.366205,"highlight":{"content":["umowy międzynarodowej i nie wypełnia przesłanek określonych w art. 89
     ust. 1 lub art. 90 <em>Konstytucji</em>","międzynarodowej lub załącznika nie
     wypełnia przesłanek określonych w art. 89 ust. 1 lub art. 90 <em>Konstytucji</em>","co do zasadności wyboru
  trybu ratyfikacji umowy międzynarodowej, o którym mowa w art. 89 ust. 2
  <em>Konstytucji</em>"]}},
{"_id":"1997_604","_score":7.10154,"highlight":{"content":["Jeżeli Trybunał Konstytucyjny wyda orzeczenie o sprzeczności celów partii 
   politycznej z <em>Konstytucją</em>","Jeżeli Trybunał Konstytucyjny wyda orzeczenie o sprzeczności z <em>Konstytucją</em>
   celów lub działalności","Ciężar udowodnienia niezgodności z <em>Konstytucją</em> spoczywa
                na wnioskodawcy, który w tym"]}},
{"_id":"1996_350","_score":7.1004834,"highlight":{"content":["Za naruszenie <em>Konstytucji</em> lub ustawy, w związku z zajmowanym
              stanowiskiem lub w zakresie","W zakresie określonym w art. 107 <em>Konstytucji</em> odpowiedzialność przed
           Trybunałem Stanu ponoszą","Członkowie Rady Ministrów ponoszą odpowiedzialność przed Trybunałem
           Stanu za naruszenie <em>Konstytucji</em>"]}},
{"_id":"1997_642","_score":6.9129972,"highlight":{"content":["wnioskami o:
             1) stwierdzenie zgodności ustaw i umów międzynarodowych z
               <em>Konstytucją</em>","stwierdzenie zgodności przepisów prawa wydawanych przez
               centralne organy państwowe, z <em>Konstytucją</em>","ratyfikowanymi
               umowami międzynarodowymi i ustawami,
             4) stwierdzenie zgodności z <em>Konstytucją</em>"]}},
{"_id":"1997_629","_score":6.5681167,"highlight":{"content":["o zmianie ustawy konstytucyjnej o trybie przygotowania
           i uchwalenia <em>Konstytucji</em> Rzeczypospolitej","W ustawie  konstytucyjnej z  dnia 23 kwietnia 1992 r. o trybie przygotowania i 
uchwalenia <em>Konstytucji</em>","Do zgłoszenia projektu <em>Konstytucji</em> załącza się wykaz 
                obywateli popierających zgłoszenie"]}},
{"_id":"1999_688","_score":6.2509775,"highlight":{"content":["postępowania w sprawie wykonywania inicjatywy
ustawodawczej przez obywateli, o której mowa w art. 118 ust. 2 <em>Konstytucji</em>","Projekt ustawy nie może dotyczyć spraw, dla których <em>Konstytucja</em>
Rzeczypospolitej Polskiej zastrzega wyłączną","Projekt ustawy wniesiony do Marszałka Sejmu powinien odpowiadać wymogom
  zawartym w <em>Konstytucji</em> i Regulaminie"]}},
{"_id":"1996_199","_score":5.62988,"highlight":{"content":["2c i art. 9-11 ustawy konstytucyjnej z dnia 23 kwietnia 
1992 r. o trybie przygotowania i uchwalenia <em>Konstytucji</em>","Prezydent Rzeczypospolitej Polskiej zarządza poddanie <em>Konstytucji</em> pod referendum
   w trybie określonym","Przyjęcie w referendum <em>Konstytucji</em> następuje wówczas, gdy opowiedziała 
   się za nią większość biorących"]}},
{"_id":"1997_681","_score":5.360203,"highlight":{"content":["Rzecznik Praw Dziecka, zwany dalej Rzecznikiem, stoi na straży praw dziecka
  określonych w <em>Konstytucji</em>","uroczyście, że przy wykonywaniu powierzonych mi obowiązków
     Rzecznika Praw Dziecka dochowam wierności <em>Konstytucji</em>"]}},
{"_id":"2001_23","_score":5.318322,"highlight":{"content":["W Dzienniku Ustaw Rzeczypospolitej Polskiej, zwanym dalej "Dziennikiem
  Ustaw", ogłasza się:
   1) <em>Konstytucję</em>","akty prawne dotyczące:
   1) stanu wojny i zawarcia pokoju,
   2) referendum zatwierdzającego zmianę <em>Konstytucji</em>","ministra, któremu Sejm wyraził wotum nieufności,
     h) powoływania lub odwoływania na określone w <em>Konstytucji</em>"]}},
{"_id":"2001_247","_score":5.0927725,"highlight":{"content":["wobec kandydatów na
         członków Rady Ministrów powoływanych w trybie art. 154 i 155
         <em>Konstytucji</em>","stanowiska, o których mowa w ust. 6, z
       wyłączeniem kandydatów powoływanych w trybie art. 154 i 155 <em>Konstytucji</em>"]}}]
```
