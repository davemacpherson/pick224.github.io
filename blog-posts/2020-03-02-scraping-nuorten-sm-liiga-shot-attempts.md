Scraping Nuorten-SM Liiga shot attempts
=======================================

Yesterday, I updated [pick224.com/2020](http://pick224.com/2020) to include stats for February’s games. After doing so, I tweeted about the update and included a screenshot showing first-time draft-eligible players sorted by shots on goal:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">I just updated <a href="https://t.co/YJiri2jTMg">https://t.co/YJiri2jTMg</a> to include stats from all of February&#39;s games.<br><br>It appears that Veeti Miettinen has been credited with 193 more shots than any other first-time draft-eligible player: <a href="https://t.co/H0UY6W1Vfl">pic.twitter.com/H0UY6W1Vfl</a></p>&mdash; Dave MacPherson (@davemacp) <a href="https://twitter.com/davemacp/status/1234257330851713028?ref_src=twsrc%5Etfw">March 1, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Finnish forward Veeti Miettinen, who is committed to join St. Cloud State University’s team next season, appeared to have staggering shot totals.

This led me to dig into the data, because it didn’t seem likely that he actually had nearly 200 more shots than anyone else. I started by looking up the game in which he was credited with the most shots, 18 in [this game](http://www.tilastopalvelu.fi/ih/gamesummary/?gid=53715&lang=en):

![alt text](18-shot-game.png "Miettinen 18 shots")

Miettinen’s team, K-Espoo, was shut out by KooKoo goalie Kasper Lehikoinen in that game. Lehikoinen was credited with 51 saves, despite K-Espoo appearing to have 75 shots. The mystery continues.

I then found the [shot map for the game](http://www.tilastopalvelu.fi/ih/gameshootingmap/?gid=53715&lang=en&season=2020), and realized the Finnish junior league tracks all shot attempts, including those that were missed or blocked:

![alt text](75-shot-attempts.png "Finnish junior shot attempts")

Cool. So, if I wanted to get actual shot on goal totals, I would need to scrape all the individual shots from the shot maps for each game.

To find how and where the shots are stored, I opened the shot map in Firefox, right-clicked, and clicked “Inspect Element” to bring up Firefox’s handy Inspector tool.

Navigating to the _Network_ tab and refreshing the page brought up all the data that goes into the shot map:

![alt text](firefox-network-finn.png "Firefox Network tab")

By clicking each of these files and looking at the _Response_ tab, I could see what data was sent in order to create the shot map. _getshootings.php_ is the file containing the data for each shot in JSON format:

![alt text](firefox-response-finn.png "Firefox Response tab")

The _Params_ tab includes the parameters the site uses to populate the _Response_ with the specific game’s data. In this case, each shot map needs a GameID and a season:

![alt text](firefox-params-finn.png "Firefox parameters")

Going back to the _Network_ tab, we can access the full _getshootings.php_ file by right-clicking and selecting “Open in New Tab”. The result is one big dictionary containing all of the shot data:

![alt text](firefox-getshootings.png "getshootings PHP file")

Unlike a lot of PHP webpages, the GameID and season don’t show in the URL — they are instead sent as hidden parameters.

Using the following Python code, I was able to pass the Game ID and season to the PHP file and extract all of the shot data to a CSV file:

<script src="https://gist.github.com/davemacpherson/4044b21454857b3860ca32a50c90a553.js"></script>

![alt text](finn-shot-attempts-csv.png "Finnish junior shot attempts CSV output")

I then adapted the code to pull every game of the season, totaled the shot attempts up by game and player, and joined each player’s shots on goal up with the rest of my data. The results make a lot more sense:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Turns out the Finnish junior league tracks all shot attempts, so Veeti Miettinen&#39;s previous totals included 148 missed and 28 blocked shots. He still holds the lead, but by much less: <a href="https://t.co/Efs677njtS">pic.twitter.com/Efs677njtS</a></p>&mdash; Dave MacPherson (@davemacp) <a href="https://twitter.com/davemacp/status/1234627343068647424?ref_src=twsrc%5Etfw">March 2, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

* * *

_If you’d like to play with some nice, clean prospect data, check out_ [_pick224.com_](http://pick224.com)_. Follow_ [_davemacp_](https://twitter.com/davemacp/) _on Twitter for updates!_
