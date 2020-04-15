Scraping Canadian Hockey League game data
=========================================

In July 2019, I launched [Pick224](https://pick224.com/), a website containing data from over 15 hockey leagues across the world with a focus on junior hockey. To compile this data, I first scrape all the individual game summaries of each league, then clean the data, total up all the stats, and join the results with each player’s biographical information.

This week, I rebuilt my Canadian Hockey League (CHL) scrapers from scratch. In what follows, I’ll describe how I found the data I needed and how I coded the scrapers in Python.

I will not be sharing the previous versions of my code in this article, for they are wretched and incoherent. But, the code worked. If you’re new to coding web scrapers or are just starting a project, I’d like to stress the importance of writing code that gets you the data you need, even if your code is inefficient and your data requires a lot of cleaning. Then, as you slowly begin to better understand your data and your needs, make small improvements. Eventually, you’ll have a great grasp of your data and some decent code.

* * *

**Understanding the data**

Before I start writing a scraper, I always want to know what data is available and what I need. For the CHL, I needed the following:  
• A listing of every goal scored in each game; and  
• A listing of every team’s lineup for each game

In this article, I’ll only dive into extracting the lineups.

I found the data I needed on one page for each game on the league’s official website. As an example, [here’s](https://ontariohockeyleague.com/gamecentre/24633/boxscore) a game summary for a recent OHL game.

The game’s lineups are listed in the “Game Summary” tab:

![alt text](chl-game-summary-tab "CHL Game Summary tab")

I could have scraped all the text from this page’s source to get what I needed, but there’s a much more organized way to get the data.

To find how and where the data is stored, I opened the game summary in Firefox, right-clicked, and clicked “Inspect Element” to bring up Firefox’s handy Inspector tool.

Navigating to the _Network_ tab, refreshing the page, and filtering for “XHR” brought up all the data that goes into the webpage:

![alt text](chl-firefox-network.png "Firefox Network tab")

By clicking each of these files and looking at the _Response_ tab, I could see what data was sent in order to create the lineups. The file ending in _tab=gamesummary_ is the file containing the data for each team’s lineup in a clean JSON tree:

![alt text](chl-firefox-response.png "Firefox Response")

The _Params_ tab includes the parameters the site uses to populate the _Response_ with the specific game’s data. In this case, the most important parameter is the “game\_id”, which is unique for each game:

![alt text](chl-firefox-params.png "Firefox parameters")

Going back to the _Network_ tab, I accessed the full _gamesummary_ file by right-clicking and selecting “Open in New Tab”. The result is one big dictionary containing all of the game’s data:

![alt text](chl-firefox-json.png "Firefox JSON")

All of the parameters above become part of the URL: [https://cluster.leaguestat.com/feed/index.php?feed=gc&key=2976319eb44abe94&client\_code=ohl&game\_id=24633&lang\_code=en&fmt=json&tab=gamesummary](https://cluster.leaguestat.com/feed/index.php?feed=gc&key=2976319eb44abe94&client_code=ohl&game_id=24633&lang_code=en&fmt=json&tab=gamesummary)

My scraper needed to pull the lineups from these pages, and I then had to organize the data in a readable format.

* * *

**Writing the Python code**

Now that I knew where to find the data I needed, it was time to write the Python code to get that data.

First, I imported the different Python libraries I would be using and specified a file to write the results to:

![alt text](chl-py-imports.png "Python imports")

Next, I specified the game\_id for the game I was scraping, along with the URL I was pulling the data from. I used Python’s _requests_ library to send a request to the specified URL so I could grab the data:

![alt text](chl-py-url.png "Python URLs")

The next step is to pull the JSON from the webpage:

![alt text](chl-py-fjson.png "Python JSON")

This code pulls the entire JSON code we saw in Firefox above:

![alt text](chl-firefox-response2 "Firefox JSON")

The JSON contains a _lot_ of data about the game:

![alt text](chl-json-ex.png "JSON example")

The home team’s lineup is found in the path _GC -> Gamesummary -> home\_team\_lineup -> players_ and the away team’s in _GC -> Gamesummary -> visitor\_team\_lineup -> players_. This code extracts only those lineups:

![alt text](chl-hdata-adata "Python lineups")

And this code converts the lineups to a much more readable format, a DataFrame, using Python’s _pandas_ library:

![alt text](chl-dfh-dfa "Python lineup DataFrame")

When printed, the home team lineup DataFrame is a table containing columns for all the different available data and rows for each player:

![alt text](chl-py-dataframe-output.png "Python DataFrame output")

The DataFrame does not include the game\_id, which would be very useful when I was ready to pull more than one game. This code adds a column containing the game\_id and a column specifying whether each player was on the home or away team:

![alt text](chl-py-add-game-no.png "Python add game number")

I then used this code to specify which columns I wanted to keep and in what order:

![alt text](chl-py-column-order.png "Python DataFrame column order")

Here’s the resulting DataFrame for the home team’s lineup:

![alt text](chl-dataframe-output-home.png "Home team lineup")

This looks good! Lastly, I wrote each lineup to the CSV file I specified in the first lines of the code, including the column headers only in the first line so they don’t show up twice:

![alt text](chl-output.png "Python output")

The result is a tidy pipe-delimited CSV file containing all the information I wanted from the game’s lineups:

![alt text](chl-output-final.png "CHL roster final output")

Here’s all the code at once:

<script src="https://gist.github.com/davemacpherson/79682783e8f2cea94ba1b853db667911.js"></script>

After I had working code for pulling one game’s data, I found the game\_ids for each game and optimized my code to pull every game in one go. I then pulled all of the goal data from the same JSON code, equipping me with everything I needed to compile the stats found on [Pick224](http://pick224.com/2020).

* * *

_If web scraping isn’t for you and you’d rather jump right into data analysis, feel free to download any of the data available on_ [_Pick224_](https://pick224.com/index.html)_. If you do something cool with the data, please let me know on_ [_Twitter_](http://twitter.com/davemacp)_!_
