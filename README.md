This repo contains some things I did to try to archive Bernie Sanders Instagram using browsertrix-crawler.

First I created a profile for crawling Instagram, which requires a login whether the site is public or not:

```
docker run -p 9222:9222 -p 9223:9223 -v $PWD:/output/ -it webrecorder/browsertrix-crawler create-login-profile --url "https://www.instagram.com/" --interactive
```

Opening http:localhost:9223 in my Chrome browser then let me login to Instagram and even to complete two-factor-authentication with my phone. When I was done clicked the *Create Profile* button at the top of the screen which wrote a `profile.tar.gz` file to my current working directory. I created a profiles directory and moved it in there:

```
mkdir profiles
mv profile.tar.gz profiles/instagram.tar.gz
```

I haven't included my profile in this GitHub repository because it contains secrets that could be used to login to my account. The recommendation is to create a "burner" account instead of using your personal one. The idea is that this profile can be kept separate from the web archive itself, which could be made public.

Then I ran the crawl using my profile:

```
docker run -p 9037:9037 -v $PWD:/crawls/ -it webrecorder/browsertrix-crawler crawl --url https://www.instagram.com/berniesanders --limit 1 --generateWACZ --text --collection berniesanders --behaviors autoscroll,siteSpecific --profile /crawls/profiles/instagram.tar.gz --screencastPort 9037
```

A few things to note here are:

- `--limit: 1` which makes sure that only that one page is crawled and that it doesn't get snarled up looking at the millions of followers of the account.
- `--behaviors autoscroll,siteSpecific` to auto-scoll the page and use the Instagram specific [browsertrix-behaviors](https://github.com/webrecorder/browsertrix-behaviors) to click to get details and comments for each post.
- `--generateWACZ` which will write a WACZ file for the crawl once finished, which is a package of the archived content and its index.
- `--screencastPort 9037` which lets you open http://localhost:9037 and watch the activity of the crawler

<img width="800" src="https://raw.githubusercontent.com/edsu/berniesanders-instagram/main/images/screencast.gif" title="Browsertrix Screencast"/>

Once the crawl completes I went to https://replayweb.page and loaded the WACZ file that is located in `collections/berniesanders/berniesanders.wacz`.

<img width="800" src="https://raw.githubusercontent.com/edsu/berniesanders-instagram/main/images/screencast.gif" title="ReplayWeb.Page Screenshot" />
