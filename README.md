This repo contains some things I did to try to archive [Bernie Sanders' Instagram](https://www.instagram.com/berniesanders) using [browsertrix-crawler](https://github.com/webrecorder/browsertrix-crawler). You will need a working [Docker environment](https://www.docker.com/) to follow along.

First I created a profile for crawling Instagram, which requires a login whether the account is public or not (in this case Bernie's is):

```bash
$ docker run -p 9222:9222 -p 9223:9223 -v $PWD:/output/ -it webrecorder/browsertrix-crawler create-login-profile --url "https://www.instagram.com/" --interactive
```

Opening `http:localhost:9223` in Chrome then let me log in to Instagram, and even to complete two-factor-authentication with my phone. When I was done I clicked the *Create Profile* button at the top of the screen which wrote a `profile.tar.gz` file to my current working directory. I then created a profiles directory and moved it in there, giving it a new name to remind me that it has my Instagram account credentials:

```bash
$ mkdir profiles
$ mv profile.tar.gz profiles/instagram.tar.gz
```

I haven't included the profile in this GitHub repository because it contains secrets that could be used to login to my account. The recommended practice is to create a "burner" account instead of using your personal one. The idea is that this profile is kept separate from the web archive itself, which could be made public.

Then I ran the crawl using my profile:

```bash
docker run -p 9037:9037 -v $PWD:/crawls/ -it webrecorder/browsertrix-crawler crawl --url https://www.instagram.com/berniesanders --limit 1 --generateWACZ --text --collection berniesanders --behaviors autoscroll,siteSpecific --profile /crawls/profiles/instagram.tar.gz --screencastPort 9037 --timeout 1000000 --behaviorTimeout 0 --scopeType page
```

A few things to note here are:

- `--profile /crawls/profiles/instagram.tar.gz` which tells browsertrix-crawler to use the Instagram profile I just created
- `--timeout 1000000` this sets a per-page timeout to make sure that the crawl doesn't run for more than 1,000,000 seconds (11 days)
- `--behaviors autoscroll,siteSpecific` to auto-scoll the page and use the Instagram specific [browsertrix-behaviors](https://github.com/webrecorder/browsertrix-behaviors) to click to get details and comments for each post.
- `--behaviorTimeout 0` this ensures that there is no timeout for behaviors (we want to browsertrix-crawler to scroll to the end of the page)
- `--scopeType page` avoids clicking other links since we just want this one user feed and not all of Instagram
- `--generateWACZ` which will write a WACZ file for the crawl once finished, which is a package of the archived content and its index.
- `--screencastPort 9037` which let me open http://localhost:9037 to watch the activity of the crawler

<img width="800" src="https://raw.githubusercontent.com/edsu/berniesanders-instagram/main/images/screencast.gif" title="Browsertrix Screencast"/>

Once the crawl completed I went to https://replayweb.page and loaded the WACZ file that is located in `collections/berniesanders/berniesanders.wacz`.

<img width="800" src="https://raw.githubusercontent.com/edsu/berniesanders-instagram/main/images/screenshot.png" title="ReplayWeb.Page Screenshot" />

Since <a href="https://replayweb.page/docs/embedding">ReplayWeb.Page</a> is really just a Web Component that operates as a client side Wayback Machine you can create an HTML file that loads some JavaScript, and creates a `<replay-web-page>` element with a `source` attribute that points to where the static WACZ file is. I've done that [here](https://edsu.github.io/berniesanders-instagram/) in this repository if you look at the `index.html`.
