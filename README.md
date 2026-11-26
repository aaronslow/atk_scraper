# America's Test Kitchen Recipe Scraper 

Scrapes America's Test Kitchen website for recipes and saves PNG screenshots and/or JSON for import to a recipe manager (e.g. https://mealie.io).

## Pre-requisites 

* Google Chrome (any recent version). Selenium 4.11+ automatically finds or downloads the matching chromedriver, so the bundled `chromedriver` binaries (Chrome v111) are no longer needed. Use `--driver` only if you want to point at a specific chromedriver.

* Python 3.8+ with an environment built off of ```requirements.txt```.

* America's Test Kitchen/Cook's Country/Cook's Illustrated web subscription (or [trial](https://www.cooksillustrated.com/trial)).
  
## Apps
### ```get_recipes.py```: Grab individual recipes from a list
* ```-h, --help``` : show this help message and exit
* ```-e EMAIL, --email EMAIL``` : ATK email for login.
* ```-p PASSWORD, --password PASSWORD``` : **Single quoted** password for login. For example ```'my_password!*'```. Optional: if omitted, the password is read from the ```ATK_PASSWORD``` environment variable, or you are prompted for it (this keeps it out of your shell history).
* ```-r RECIPES, --recipes RECIPES``` : Text file containing a list of **individual recipes** to grab.
* ```-j JSON, --json JSON``` : Get recipes as json for mealie (default **True**)
* ```-i IMAGE, --image IMAGE``` : Get recipes as images (default **False**)
* ```-o OUT_PATH, --out_path OUT_PATH``` : Location to save images/json (default './recipes/')
* ```--driver DRIVER ``` : Path to a chromedriver matching your Chrome. (default: Selenium finds/downloads one)
* ```--chrome CHROME ``` : Path to the Chrome/Chromium binary. (default: system Chrome)
* ```--verbose``` : verbose output

Example:
```
python get_recipes.py -e you@example.com -r recipes_to_get.txt -o ./recipes/
```

### ```get_searches.py```: Traverse search results and grab all recipes within
* ```-h, --help``` : show this help message and exit
* ```-e EMAIL, --email EMAIL``` : ATK email for login.
* ```-p PASSWORD, --password PASSWORD``` : **Single quoted** ATK password for login. For example ```'my_password!*'```. Optional: if omitted, the password is read from the ```ATK_PASSWORD``` environment variable, or you are prompted for it.
* ```-r RECIPES, --recipes RECIPES``` : Text file containing a list of pages to grab all recipes from. See recipes.txt for an example. Two kinds of page work:
  * **Search pages** (```https://www.americastestkitchen.com/search?...```). Search on the site, tick the filters you want, and copy the URL. All results are loaded, not just the first page. Old-style URLs with ```fR[...]``` filters are converted automatically. The site may stop returning results for very large searches, so split those into smaller filtered searches.
  * **Cookbook and collection pages** (```/books/...```, ```/collections/...```). The site now shows only a cookbook's **featured recipes** (about 30), and search can no longer be filtered by cookbook, so a whole book can't be fetched anymore.
* ```-j JSON, --json JSON``` : Get recipes as json for mealie (default **True**)
* ```-i IMAGE, --image IMAGE``` : Get recipes as images (default **False**)
* ```--sortby SORTBY``` : Order to load search results in: ```popularity``` or ```rating``` (Highest Rated), ```date``` (Most Recent) or ```relevance``` (default **popularity**). Only applied to searches without search words: for searches with words, the site's sorted views return only close matches (e.g. 18 instead of 203 results), so those stay in relevance order.
* ```-o OUT_PATH, --out_path OUT_PATH``` : Location to save images/json (default ``./recipes/``)
* ```--driver DRIVER ``` : Path to a chromedriver matching your Chrome. (default: Selenium finds/downloads one)
* ```--chrome CHROME ``` : Path to the Chrome/Chromium binary. (default: system Chrome)
* ```--verbose``` : verbose output

Recipes that were already saved to the output folder are skipped, so an interrupted run can be restarted.

## Process
1. Selenium opens Chrome driver in headless mode.
2. Logs into ATK using credentials provided.
3. Iterates through the list of pages, whether individual recipes or full search pages.
5. For search pages, all results are loaded (the site's "Load More" adds pages to the URL as ```p=N```, so the script loads a large enough ```p```). The page source is passed to BeautifulSoup, which extracts all recipe links.
6. Each recipe link is loaded with Selenium. Page dimensions are determined using page divs. 
7. If ```-i``` is specified, a screenshot is saved. The Chrome window is resized to fit these dimensions and a screenshot is saved to the specified path. Screenshots are cleaned using Pillow and saved as ```<image>.trimmed.png```
8. If ```-j``` is specified, recipe information is read from the schema.org Recipe data ATK embeds in each page (plus ingredient group headings and "Why This Recipe Works" from the page itself) and loaded into JSON for later import to a recipe manager (e.g. mealie). The highlight image is also saved as a ```.jp2``` image (this is the format used by ATK)
9. The program will load the next page and repeat.

## Disclaimer

This project is for educational, read-only purposes.

The use of this project is done at your own discretion and risk. 

You are solely responsible for liability and consequences.


