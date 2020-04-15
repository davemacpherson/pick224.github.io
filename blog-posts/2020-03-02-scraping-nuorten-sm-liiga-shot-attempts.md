Scraping Nuorten-SM Liiga shot attempts
=======================================

[![Dave MacPherson](https://miro.medium.com/fit/c/96/96/0*vQ6OsbHjqiKNB_6i.jpg)](https://medium.com/@dmacp?source=post_page-----ae67c1800fe7----------------------)[Dave MacPherson](https://medium.com/@dmacp?source=post_page-----ae67c1800fe7----------------------)Follow[Mar 3](https://medium.com/@dmacp/scraping-nuorten-sm-liiga-shot-attempts-ae67c1800fe7?source=post_page-----ae67c1800fe7----------------------) · 3 min read

Yesterday, I updated [pick224.com/2020](http://pick224.com/2020) to include stats for February’s games. After doing so, I tweeted about the update and included a screenshot showing first-time draft-eligible players sorted by shots on goal:

Finnish forward Veeti Miettinen, who is committed to join St. Cloud State University’s team next season, appeared to have staggering shot totals.

This led me to dig into the data, because it didn’t seem likely that he actually had nearly 200 more shots than anyone else. I started by looking up the game in which he was credited with the most shots, 18 in [this game](http://www.tilastopalvelu.fi/ih/gamesummary/?gid=53715&lang=en):

<img class="s t u gs ai" src="https://miro.medium.com/max/2768/1\*\_H3\_hRVdSgXhCY9chy6kYw.png" width="1384" height="1072" srcSet="https://miro.medium.com/max/552/1\*\_H3\_hRVdSgXhCY9chy6kYw.png 276w, https://miro.medium.com/max/1104/1\*\_H3\_hRVdSgXhCY9chy6kYw.png 552w, https://miro.medium.com/max/1280/1\*\_H3\_hRVdSgXhCY9chy6kYw.png 640w, https://miro.medium.com/max/1400/1\*\_H3\_hRVdSgXhCY9chy6kYw.png 700w" sizes="700px" role="presentation"/>

Miettinen’s team, K-Espoo, was shut out by KooKoo goalie Kasper Lehikoinen in that game. Lehikoinen was credited with 51 saves, despite K-Espoo appearing to have 75 shots. The mystery continues.

I then found the [shot map for the game](http://www.tilastopalvelu.fi/ih/gameshootingmap/?gid=53715&lang=en&season=2020), and realized the Finnish junior league tracks all shot attempts, including those that were missed or blocked:

<img class="s t u gs ai" src="https://miro.medium.com/max/1784/1\*Ji0qTsfHixsc3\_Qu7Ej03w.png" width="892" height="246" srcSet="https://miro.medium.com/max/552/1\*Ji0qTsfHixsc3\_Qu7Ej03w.png 276w, https://miro.medium.com/max/1104/1\*Ji0qTsfHixsc3\_Qu7Ej03w.png 552w, https://miro.medium.com/max/1280/1\*Ji0qTsfHixsc3\_Qu7Ej03w.png 640w, https://miro.medium.com/max/1400/1\*Ji0qTsfHixsc3\_Qu7Ej03w.png 700w" sizes="700px" role="presentation"/>

Cool. So, if I wanted to get actual shot on goal totals, I would need to scrape all the individual shots from the shot maps for each game.

To find how and where the shots are stored, I opened the shot map in Firefox, right-clicked, and clicked “Inspect Element” to bring up Firefox’s handy Inspector tool.

Navigating to the _Network_ tab and refreshing the page brought up all the data that goes into the shot map:

<img class="s t u gs ai" src="https://miro.medium.com/max/2040/1\*dQOio00au6LWpj73h3JzdQ.png" width="1020" height="758" srcSet="https://miro.medium.com/max/552/1\*dQOio00au6LWpj73h3JzdQ.png 276w, https://miro.medium.com/max/1104/1\*dQOio00au6LWpj73h3JzdQ.png 552w, https://miro.medium.com/max/1280/1\*dQOio00au6LWpj73h3JzdQ.png 640w, https://miro.medium.com/max/1400/1\*dQOio00au6LWpj73h3JzdQ.png 700w" sizes="700px" role="presentation"/>

By clicking each of these files and looking at the _Response_ tab, I could see what data was sent in order to create the shot map. _getshootings.php_ is the file containing the data for each shot in JSON format:

<img class="s t u gs ai" src="https://miro.medium.com/max/1840/1\*AWqjV5kmZfowD44X-Rbpsg.png" width="920" height="908" srcSet="https://miro.medium.com/max/552/1\*AWqjV5kmZfowD44X-Rbpsg.png 276w, https://miro.medium.com/max/1104/1\*AWqjV5kmZfowD44X-Rbpsg.png 552w, https://miro.medium.com/max/1280/1\*AWqjV5kmZfowD44X-Rbpsg.png 640w, https://miro.medium.com/max/1400/1\*AWqjV5kmZfowD44X-Rbpsg.png 700w" sizes="700px" role="presentation"/>

The _Params_ tab includes the parameters the site uses to populate the _Response_ with the specific game’s data. In this case, each shot map needs a GameID and a season:

<img class="s t u gs ai" src="https://miro.medium.com/max/1852/1\*AkRLRnIMgrQ-bdJo\_\_uuRw.png" width="926" height="320" srcSet="https://miro.medium.com/max/552/1\*AkRLRnIMgrQ-bdJo\_\_uuRw.png 276w, https://miro.medium.com/max/1104/1\*AkRLRnIMgrQ-bdJo\_\_uuRw.png 552w, https://miro.medium.com/max/1280/1\*AkRLRnIMgrQ-bdJo\_\_uuRw.png 640w, https://miro.medium.com/max/1400/1\*AkRLRnIMgrQ-bdJo\_\_uuRw.png 700w" sizes="700px" role="presentation"/>

Going back to the _Network_ tab, we can access the full _getshootings.php_ file by right-clicking and selecting “Open in New Tab”. The result is one big dictionary containing all of the shot data:

<img class="s t u gs ai" src="https://miro.medium.com/max/4756/1\*ny9jv2NjdjMEyjKq0XMsbg.png" width="2378" height="374" srcSet="https://miro.medium.com/max/552/1\*ny9jv2NjdjMEyjKq0XMsbg.png 276w, https://miro.medium.com/max/1104/1\*ny9jv2NjdjMEyjKq0XMsbg.png 552w, https://miro.medium.com/max/1280/1\*ny9jv2NjdjMEyjKq0XMsbg.png 640w, https://miro.medium.com/max/1400/1\*ny9jv2NjdjMEyjKq0XMsbg.png 700w" sizes="700px" role="presentation"/>

Unlike a lot of PHP webpages, the GameID and season don’t show in the URL — they are instead sent as hidden parameters.

Using the following Python code, I was able to pass the Game ID and season to the PHP file and extract all of the shot data to a CSV file:

<img class="s t u gs ai" src="https://miro.medium.com/max/4988/1\*uxd5g1YAKBim99vaSQvIoA.png" width="2494" height="454" srcSet="https://miro.medium.com/max/552/1\*uxd5g1YAKBim99vaSQvIoA.png 276w, https://miro.medium.com/max/1104/1\*uxd5g1YAKBim99vaSQvIoA.png 552w, https://miro.medium.com/max/1280/1\*uxd5g1YAKBim99vaSQvIoA.png 640w, https://miro.medium.com/max/1400/1\*uxd5g1YAKBim99vaSQvIoA.png 700w" sizes="700px" role="presentation"/>

The resulting CSV file

I then adapted the code to pull every game of the season, totalled the shot attempts up by game and player, and joined each player’s shots on goal up with the rest of my data. The results make a lot more sense:

* * *

_If you’d like to play with some nice, clean prospect data, check out_ [_pick224.com_](http://pick224.com)_. Follow_ [_davemacp_](https://twitter.com/davemacp/) _on Twitter for updates!_
