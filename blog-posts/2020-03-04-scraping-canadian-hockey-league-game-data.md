Scraping Canadian Hockey League game data
=========================================

[![Dave MacPherson](https://miro.medium.com/fit/c/96/96/0*vQ6OsbHjqiKNB_6i.jpg)](https://medium.com/@dmacp?source=post_page-----3472dceadaea----------------------)[Dave MacPherson](https://medium.com/@dmacp?source=post_page-----3472dceadaea----------------------)Follow[Mar 4](https://medium.com/@dmacp/scraping-canadian-hockey-league-game-data-3472dceadaea?source=post_page-----3472dceadaea----------------------) · 6 min read

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

<img class="s t u hh ai" src="https://miro.medium.com/max/3856/1\*Kcyrk7H7wYHSWYLYqV4sFg.png" width="1928" height="918" srcSet="https://miro.medium.com/max/552/1\*Kcyrk7H7wYHSWYLYqV4sFg.png 276w, https://miro.medium.com/max/1104/1\*Kcyrk7H7wYHSWYLYqV4sFg.png 552w, https://miro.medium.com/max/1280/1\*Kcyrk7H7wYHSWYLYqV4sFg.png 640w, https://miro.medium.com/max/1400/1\*Kcyrk7H7wYHSWYLYqV4sFg.png 700w" sizes="700px" role="presentation"/>

I could have scraped all the text from this page’s source to get what I needed, but there’s a much more organized way to get the data.

To find how and where the data is stored, I opened the game summary in Firefox, right-clicked, and clicked “Inspect Element” to bring up Firefox’s handy Inspector tool.

Navigating to the _Network_ tab, refreshing the page, and filtering for “XHR” brought up all the data that goes into the webpage:

<img class="s t u hh ai" src="https://miro.medium.com/max/2628/1\*bTCxnYeg5SZx2p9xer\_XoA.png" width="1314" height="670" srcSet="https://miro.medium.com/max/552/1\*bTCxnYeg5SZx2p9xer\_XoA.png 276w, https://miro.medium.com/max/1104/1\*bTCxnYeg5SZx2p9xer\_XoA.png 552w, https://miro.medium.com/max/1280/1\*bTCxnYeg5SZx2p9xer\_XoA.png 640w, https://miro.medium.com/max/1400/1\*bTCxnYeg5SZx2p9xer\_XoA.png 700w" sizes="700px" role="presentation"/>

By clicking each of these files and looking at the _Response_ tab, I could see what data was sent in order to create the lineups. The file ending in _tab=gamesummary_ is the file containing the data for each team’s lineup in a clean JSON tree:

<img class="s t u hh ai" src="https://miro.medium.com/max/1940/1\*74wRF4e2UHWhDWU\_tOuMLw.png" width="970" height="1094" srcSet="https://miro.medium.com/max/552/1\*74wRF4e2UHWhDWU\_tOuMLw.png 276w, https://miro.medium.com/max/1104/1\*74wRF4e2UHWhDWU\_tOuMLw.png 552w, https://miro.medium.com/max/1280/1\*74wRF4e2UHWhDWU\_tOuMLw.png 640w, https://miro.medium.com/max/1400/1\*74wRF4e2UHWhDWU\_tOuMLw.png 700w" sizes="700px" role="presentation"/>

The _Params_ tab includes the parameters the site uses to populate the _Response_ with the specific game’s data. In this case, the most important parameter is the “game\_id”, which is unique for each game:

<img class="s t u hh ai" src="https://miro.medium.com/max/944/1\*gmYEP9BbIKKFrKx-phJW\_Q.png" width="472" height="438" srcSet="https://miro.medium.com/max/552/1\*gmYEP9BbIKKFrKx-phJW\_Q.png 276w, https://miro.medium.com/max/944/1\*gmYEP9BbIKKFrKx-phJW\_Q.png 472w" sizes="472px" role="presentation"/>

Going back to the _Network_ tab, I accessed the full _gamesummary_ file by right-clicking and selecting “Open in New Tab”. The result is one big dictionary containing all of the game’s data:

<img class="s t u hh ai" src="https://miro.medium.com/max/6612/1\*4U\_kA8TPc3KKY4RJt1jcxg.png" width="3306" height="498" srcSet="https://miro.medium.com/max/552/1\*4U\_kA8TPc3KKY4RJt1jcxg.png 276w, https://miro.medium.com/max/1104/1\*4U\_kA8TPc3KKY4RJt1jcxg.png 552w, https://miro.medium.com/max/1280/1\*4U\_kA8TPc3KKY4RJt1jcxg.png 640w, https://miro.medium.com/max/1456/1\*4U\_kA8TPc3KKY4RJt1jcxg.png 728w, https://miro.medium.com/max/1632/1\*4U\_kA8TPc3KKY4RJt1jcxg.png 816w, https://miro.medium.com/max/1808/1\*4U\_kA8TPc3KKY4RJt1jcxg.png 904w, https://miro.medium.com/max/1984/1\*4U\_kA8TPc3KKY4RJt1jcxg.png 992w, https://miro.medium.com/max/2000/1\*4U\_kA8TPc3KKY4RJt1jcxg.png 1000w" sizes="1000px" role="presentation"/>

All of the parameters above become part of the URL: [https://cluster.leaguestat.com/feed/index.php?feed=gc&key=2976319eb44abe94&client\_code=ohl&game\_id=24633&lang\_code=en&fmt=json&tab=gamesummary](https://cluster.leaguestat.com/feed/index.php?feed=gc&key=2976319eb44abe94&client_code=ohl&game_id=24633&lang_code=en&fmt=json&tab=gamesummary)

My scraper needed to pull the lineups from these pages, and I then had to organize the data in a readable format.

* * *

**Writing the Python code**

Now that I knew where to find the data I needed, it was time to write the Python code to get that data.

First, I imported the different Python libraries I would be using and specified a file to write the results to:

<img class="s t u hh ai" src="https://miro.medium.com/max/1476/1\*\_o5p3X04k234giB-EYHEag.png" width="738" height="292" srcSet="https://miro.medium.com/max/552/1\*\_o5p3X04k234giB-EYHEag.png 276w, https://miro.medium.com/max/1104/1\*\_o5p3X04k234giB-EYHEag.png 552w, https://miro.medium.com/max/1280/1\*\_o5p3X04k234giB-EYHEag.png 640w, https://miro.medium.com/max/1400/1\*\_o5p3X04k234giB-EYHEag.png 700w" sizes="700px" role="presentation"/>

Next, I specified the game\_id for the game I was scraping, along with the URL I was pulling the data from. I used Python’s _requests_ library to send a request to the specified URL so I could grab the data:

<img class="s t u hh ai" src="https://miro.medium.com/max/5476/1\*UNbTBkeXh3Gba9TjRv1fog.png" width="2738" height="396" srcSet="https://miro.medium.com/max/552/1\*UNbTBkeXh3Gba9TjRv1fog.png 276w, https://miro.medium.com/max/1104/1\*UNbTBkeXh3Gba9TjRv1fog.png 552w, https://miro.medium.com/max/1280/1\*UNbTBkeXh3Gba9TjRv1fog.png 640w, https://miro.medium.com/max/1456/1\*UNbTBkeXh3Gba9TjRv1fog.png 728w, https://miro.medium.com/max/1632/1\*UNbTBkeXh3Gba9TjRv1fog.png 816w, https://miro.medium.com/max/1808/1\*UNbTBkeXh3Gba9TjRv1fog.png 904w, https://miro.medium.com/max/1984/1\*UNbTBkeXh3Gba9TjRv1fog.png 992w, https://miro.medium.com/max/2000/1\*UNbTBkeXh3Gba9TjRv1fog.png 1000w" sizes="1000px" role="presentation"/>

The next step is to pull the JSON from the webpage:

<img class="s t u hh ai" src="https://miro.medium.com/max/1072/1\*QbMkTmcOk6Ug1-bLS7pcyQ.png" width="536" height="100" srcSet="https://miro.medium.com/max/552/1\*QbMkTmcOk6Ug1-bLS7pcyQ.png 276w, https://miro.medium.com/max/1072/1\*QbMkTmcOk6Ug1-bLS7pcyQ.png 536w" sizes="536px" role="presentation"/>

This code pulls the entire JSON code we saw in Firefox above:

<img class="s t u hh ai" src="https://miro.medium.com/max/2112/1\*i6R3lzPQkSI2whePBoR1cw.png" width="1056" height="672" srcSet="https://miro.medium.com/max/552/1\*i6R3lzPQkSI2whePBoR1cw.png 276w, https://miro.medium.com/max/1104/1\*i6R3lzPQkSI2whePBoR1cw.png 552w, https://miro.medium.com/max/1280/1\*i6R3lzPQkSI2whePBoR1cw.png 640w, https://miro.medium.com/max/1400/1\*i6R3lzPQkSI2whePBoR1cw.png 700w" sizes="700px" role="presentation"/>

The JSON contains a _lot_ of data about the game:

<img class="s t u hh ai" src="https://miro.medium.com/max/5328/1\*LwZEofGK964M\_DILNpaSgg.png" width="2664" height="504" srcSet="https://miro.medium.com/max/552/1\*LwZEofGK964M\_DILNpaSgg.png 276w, https://miro.medium.com/max/1104/1\*LwZEofGK964M\_DILNpaSgg.png 552w, https://miro.medium.com/max/1280/1\*LwZEofGK964M\_DILNpaSgg.png 640w, https://miro.medium.com/max/1400/1\*LwZEofGK964M\_DILNpaSgg.png 700w" sizes="700px" role="presentation"/>

The home team’s lineup is found in the path _GC -> Gamesummary -> home\_team\_lineup -> players_ and the away team’s in _GC -> Gamesummary -> visitor\_team\_lineup -> players_. This code extracts only those lineups:

<img class="s t u hh ai" src="https://miro.medium.com/max/2232/1\*48pO-hTjH7rMtf3w2v9ljw.png" width="1116" height="134" srcSet="https://miro.medium.com/max/552/1\*48pO-hTjH7rMtf3w2v9ljw.png 276w, https://miro.medium.com/max/1104/1\*48pO-hTjH7rMtf3w2v9ljw.png 552w, https://miro.medium.com/max/1280/1\*48pO-hTjH7rMtf3w2v9ljw.png 640w, https://miro.medium.com/max/1400/1\*48pO-hTjH7rMtf3w2v9ljw.png 700w" sizes="700px" role="presentation"/>

And this code converts the lineups to a much more readable format, a DataFrame, using Python’s _pandas_ library:

<img class="s t u hh ai" src="https://miro.medium.com/max/1456/1\*dwFLEanYEuG3BkMvHBAonw.png" width="728" height="130" srcSet="https://miro.medium.com/max/552/1\*dwFLEanYEuG3BkMvHBAonw.png 276w, https://miro.medium.com/max/1104/1\*dwFLEanYEuG3BkMvHBAonw.png 552w, https://miro.medium.com/max/1280/1\*dwFLEanYEuG3BkMvHBAonw.png 640w, https://miro.medium.com/max/1400/1\*dwFLEanYEuG3BkMvHBAonw.png 700w" sizes="700px" role="presentation"/>

When printed, the home team lineup DataFrame is a table containing columns for all the different available data and rows for each player:

<img class="s t u hh ai" src="https://miro.medium.com/max/2792/1\*rSmE7akn7H2S6FNUeFt-9Q.png" width="1396" height="330" srcSet="https://miro.medium.com/max/552/1\*rSmE7akn7H2S6FNUeFt-9Q.png 276w, https://miro.medium.com/max/1104/1\*rSmE7akn7H2S6FNUeFt-9Q.png 552w, https://miro.medium.com/max/1280/1\*rSmE7akn7H2S6FNUeFt-9Q.png 640w, https://miro.medium.com/max/1400/1\*rSmE7akn7H2S6FNUeFt-9Q.png 700w" sizes="700px" role="presentation"/>

The DataFrame does not include the game\_id, which would be very useful when I was ready to pull more than one game. This code adds a column containing the game\_id and a column specifying whether each player was on the home or away team:

<img class="s t u hh ai" src="https://miro.medium.com/max/2300/1\*nmYpzras5z\_wlUItOEa9PQ.png" width="1150" height="194" srcSet="https://miro.medium.com/max/552/1\*nmYpzras5z\_wlUItOEa9PQ.png 276w, https://miro.medium.com/max/1104/1\*nmYpzras5z\_wlUItOEa9PQ.png 552w, https://miro.medium.com/max/1280/1\*nmYpzras5z\_wlUItOEa9PQ.png 640w, https://miro.medium.com/max/1400/1\*nmYpzras5z\_wlUItOEa9PQ.png 700w" sizes="700px" role="presentation"/>

I then used this code to specify which columns I wanted to keep and in what order:

<img class="s t u hh ai" src="https://miro.medium.com/max/6584/1\*Cx5hp8n9Z5OgXrh\_B6JagQ.png" width="3292" height="170" srcSet="https://miro.medium.com/max/552/1\*Cx5hp8n9Z5OgXrh\_B6JagQ.png 276w, https://miro.medium.com/max/1104/1\*Cx5hp8n9Z5OgXrh\_B6JagQ.png 552w, https://miro.medium.com/max/1280/1\*Cx5hp8n9Z5OgXrh\_B6JagQ.png 640w, https://miro.medium.com/max/1456/1\*Cx5hp8n9Z5OgXrh\_B6JagQ.png 728w, https://miro.medium.com/max/1632/1\*Cx5hp8n9Z5OgXrh\_B6JagQ.png 816w, https://miro.medium.com/max/1808/1\*Cx5hp8n9Z5OgXrh\_B6JagQ.png 904w, https://miro.medium.com/max/1984/1\*Cx5hp8n9Z5OgXrh\_B6JagQ.png 992w, https://miro.medium.com/max/2000/1\*Cx5hp8n9Z5OgXrh\_B6JagQ.png 1000w" sizes="1000px" role="presentation"/>

Here’s the resulting DataFrame for the home team’s lineup:

<img class="s t u hh ai" src="https://miro.medium.com/max/4976/1\*Csn4h5Q8ngYH-my6x068xQ.png" width="2488" height="322" srcSet="https://miro.medium.com/max/552/1\*Csn4h5Q8ngYH-my6x068xQ.png 276w, https://miro.medium.com/max/1104/1\*Csn4h5Q8ngYH-my6x068xQ.png 552w, https://miro.medium.com/max/1280/1\*Csn4h5Q8ngYH-my6x068xQ.png 640w, https://miro.medium.com/max/1456/1\*Csn4h5Q8ngYH-my6x068xQ.png 728w, https://miro.medium.com/max/1632/1\*Csn4h5Q8ngYH-my6x068xQ.png 816w, https://miro.medium.com/max/1808/1\*Csn4h5Q8ngYH-my6x068xQ.png 904w, https://miro.medium.com/max/1984/1\*Csn4h5Q8ngYH-my6x068xQ.png 992w, https://miro.medium.com/max/2000/1\*Csn4h5Q8ngYH-my6x068xQ.png 1000w" sizes="1000px" role="presentation"/>

This looks good! Lastly, I wrote each lineup to the CSV file I specified in the first lines of the code, including the column headers only in the first line so they don’t show up twice:

<img class="s t u hh ai" src="https://miro.medium.com/max/2468/1\*\_5DoqfaE5A7bjyOlHHpPwA.png" width="1234" height="102" srcSet="https://miro.medium.com/max/552/1\*\_5DoqfaE5A7bjyOlHHpPwA.png 276w, https://miro.medium.com/max/1104/1\*\_5DoqfaE5A7bjyOlHHpPwA.png 552w, https://miro.medium.com/max/1280/1\*\_5DoqfaE5A7bjyOlHHpPwA.png 640w, https://miro.medium.com/max/1400/1\*\_5DoqfaE5A7bjyOlHHpPwA.png 700w" sizes="700px" role="presentation"/>

The result is a tidy pipe-delimited CSV file containing all the information I wanted from the game’s lineups:

<img class="s t u hh ai" src="https://miro.medium.com/max/2872/1\*TYF-s3znVEn5IEfgOZST6g.png" width="1436" height="928" srcSet="https://miro.medium.com/max/552/1\*TYF-s3znVEn5IEfgOZST6g.png 276w, https://miro.medium.com/max/1104/1\*TYF-s3znVEn5IEfgOZST6g.png 552w, https://miro.medium.com/max/1280/1\*TYF-s3znVEn5IEfgOZST6g.png 640w, https://miro.medium.com/max/1400/1\*TYF-s3znVEn5IEfgOZST6g.png 700w" sizes="700px" role="presentation"/>

Here’s all the code at once:

After I had working code for pulling one game’s data, I found the game\_ids for each game and optimized my code to pull every game in one go. I then pulled all of the goal data from the same JSON code, equipping me with everything I needed to compile the stats found on [Pick224](http://pick224.com/2020).

* * *

_If web scraping isn’t for you and you’d rather jump right into data analysis, feel free to download any of the data available on_ [_Pick224_](https://pick224.com/index.html)_. If you do something cool with the data, please let me know on_ [_Twitter_](http://twitter.com/davemacp)_!_
