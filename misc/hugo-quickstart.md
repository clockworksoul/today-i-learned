# Hugo Quickstart

_Source: [Hugo Quickstart](https://gohugo.io/getting-started/quick-start/)_

The following are the steps I used to get [Hugo](https://gohugo.io) started. They're close, but not identical, to the ones on the [official quickstart page](https://gohugo.io/getting-started/quick-start/).

## Step 1: Install Hugo

```bash
go get github.com/gohugoio/hugo
```

```bash
hugo version
```

## Step 2: Create a New Site

```bash
hugo new site very-serio.us
```

## Step 3: Add a Theme

See [themes.gohugo.io](https://themes.gohugo.io/) for a list of themes to consider. This quickstart uses the beautiful [Ananke](https://themes.gohugo.io/gohugo-theme-ananke/) theme.

First, download the theme from GitHub and add it to your site’s theme directory:

```bash
cd very-serio.us
git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
```

Then, add the theme to the site configuration:

```bash
echo 'theme = "ananke"' >> config.toml
```

## Step 4: Add Some Content 

```bash
hugo new posts/my-first-post.md
```

## Step 5: Start the Hugo server 

Now, start the Hugo server with drafts enabled:

```bash
$ hugo server -D

                   | EN
+------------------+----+
  Pages            | 10
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  3
  Processed images |  0
  Aliases          |  1
  Sitemaps         |  1
  Cleaned          |  0

Total in 11 ms
Watching for changes in /Users/bep/quickstart/{content,data,layouts,static,themes}
Watching for config changes in /Users/bep/quickstart/config.toml
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

Navigate to your new site at http://localhost:1313/.

Feel free to edit or add new content and simply refresh in browser to see changes quickly (You might need to force refresh in webbrowser, something like Ctrl-R usually works).

## Step 6: Customize the Theme 

Your new site already looks great, but you will want to tweak it a little before you release it to the public.

### Site Configuration 

Open up `config.toml` in a text editor:

```
baseURL = "https://example.org/"
languageCode = "en-us"
title = "My New Hugo Site"
theme = "ananke"
```

## Step 7: Build static pages 

It is simple. Just call:

```bash
hugo -D
```

Output will be in `./public/` directory by default (`-d`/`--destination` flag to change it, or set `publishdir` in the config file).

Drafts do not get deployed; once you finish a post, update the header of the post to say draft: `false`. More info [here](https://gohugo.io/getting-started/usage/#draft-future-and-expired-content).