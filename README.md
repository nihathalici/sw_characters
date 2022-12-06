# Searching Star Wars Characters

This exercise were part of "Select Topics in Python: Packaging" course in Coursera by Codio. 

We are going to use requests package to query a Star Wars API and print a short description of a character that looks like the output below. In addition, the script should be able to handle a list of characters returned from the API.

```
Luke Skywalker is from the planet Tatooine. They appear in the following films:
  * A New Hope
  * The Empire Strikes Back
  * Return of the Jedi
  * Revenge of the Sith
```

search_api.py
========================================================
```Python3
import requests

def search(search_term='luke'):
  base_url = 'https://swapi.dev/api/people/?search='
  search_url = f'{base_url}{search_term}'
  resp = requests.get(search_url)
  resp_json = resp.json()
  if resp_json.get('results'):
    return resp.json()['results']
  else:
    return None

def parse_name(person):
  name = person.get('name')
  return name

def parse_planet(person):
  planet_url = person.get('homeworld')
  resp = requests.get(planet_url)
  planet = resp.json().get('name')
  return planet

def parse_films(person):
  film_urls = person.get('films')
  films = [fetch_title(film_url) for film_url in film_urls]
  return films

def fetch_title(url):
  film_json = requests.get(url).json()
  film_title = film_json.get('title')
  return film_title

def format_titles(titles):
  new_lines = [title + '\n'for title in titles]
  formatted_titles = '  * ' + '  * '.join(new_lines)
  return formatted_titles

def person_description(name, planet, titles):
  description = f'{name} is from the planet {planet}. They appear in the following films:\n{titles}'
  return description

if __name__ == '__main__':
  import pprint

  person = search()
  name = parse_name(person)
  planet = parse_planet(person)
  film_list = parse_films(person)
  titles = format_titles(film_list)
  description = person_description(name, planet, titles)

  print(description)
```

interface.py
========================================================
```Python3
import fire
from tinydb import TinyDB, Query
import search_api

db = TinyDB('db.json')
User = Query()

def search(name='luke'):
  characters = search_api.search(name)
  if characters is not None:
    check_db(characters)
  else:
    print(f'Cannot find the character "{name}"')

def parse_char(char):
  char_name = search_api.parse_name(char)
  planet= search_api.parse_planet(char)
  film_list = search_api.parse_films(char)
  titles = search_api.format_titles(film_list)
  description = search_api.person_description(char_name, planet, titles)
  db.insert({'name':char_name, 'planet':planet, 'titles':titles})
  return description

def check_db(chars):
  for char in chars:
    char_name = search_api.parse_name(char)
    results = db.search(User.name == char_name)
    if not results:
      description = parse_char(char)
    else:
      name = results[0]['name']
      planet = results[0]['planet']
      titles = results[0]['titles']
      description = search_api.person_description(name, planet, titles)
    print(description)

if __name__ == '__main__':
  fire.Fire(search)
```




requirements.yaml
========================================================
```yaml

name: sw
channels:
  - conda-forge
  - defaults
dependencies:
  - _libgcc_mutex=0.1=main
  - _openmp_mutex=5.1=1_gnu
  - bzip2=1.0.8=h7b6447c_0
  - ca-certificates=2022.9.24=ha878542_0
  - certifi=2022.9.24=pyhd8ed1ab_0
  - charset-normalizer=2.0.4=pyhd3eb1b0_0
  - ld_impl_linux-64=2.38=h1181459_1
  - libffi=3.4.2=h6a678d5_6
  - libgcc-ng=11.2.0=h1234567_1
  - libgomp=11.2.0=h1234567_1
  - libstdcxx-ng=11.2.0=h1234567_1
  - libuuid=1.41.5=h5eee18b_0
  - ncurses=6.3=h5eee18b_3
  - openssl=1.1.1s=h7f8727e_0
  - pycparser=2.21=pyhd3eb1b0_0
  - pyopenssl=22.0.0=pyhd3eb1b0_0
  - python=3.10.8=h7a1cb2a_1
  - readline=8.2=h5eee18b_0
  - sqlite=3.40.0=h5082296_0
  - tinydb=4.7.0=pyhd8ed1ab_0
  - tk=8.6.12=h1ccaba5_0
  - tzdata=2022g=h04d1e81_0
  - wheel=0.37.1=pyhd3eb1b0_0
  - xz=5.2.8=h5eee18b_0
  - zlib=1.2.13=h5eee18b_0
  - pip:
    - brotlipy==0.7.0
    - cffi==1.15.1
    - cryptography==38.0.1
    - fire==0.4.0
    - idna==3.4
    - pip==22.2.2
    - pysocks==1.7.1
    - requests==2.28.1
    - setuptools==65.5.0
    - six==1.16.0
    - termcolor==2.1.1
    - urllib3==1.26.12
prefix: /home/codio/anaconda3/envs/sw
```
