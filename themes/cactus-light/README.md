# Cactus Light

A responsive, light and simple [Hexo](http://hexo.io) theme for a personal website. Based on the original [Cactus Dark](https://github.com/probberechts/cactus-dark) theme by [Pieter Robberechts](https://github.com/probberechts).

:cactus: [Demo](http://gabithu.me)

![cactus-light](https://cloud.githubusercontent.com/assets/2175271/19885143/62e9269c-a01d-11e6-8e26-e36a36201d88.png)

## Summary

- [General](#general)
- [Features](#features)
- [Install](#install)
- [Configuration](#configuration)
- [License](#license)

## General

- **Version** : 2.0
- **Compatibility** : Hexo 3 or later

## Features

- Fully responsive
- Disqus
- Googe analytics
- Font Awesome icons
- Pick your own code highlighting scheme
- Configurable navigation menu
- Projects list
- Simplicity

## Install
1. In the `root` directory:

    ```git
    $ git clone https://github.com/gabithume/cactus-light.git themes/cactus-light
    $ npm install hexo-pagination --save
    ```

2. Change the `theme` property in the `config.yml` file.

    ```yml
    # theme: landscape
    theme: cactus-light
    ```

3. Run: `hexo generate` and `hexo server`

## Configuration

### Navigation

Setup the navigation menu in the theme's `_config.yml`:

  ```
  nav:
    Home: /
    About: /about/
    Writing: /archives/
    Projects: http://github.com/gabithume
    LINK_NAME: URL
  ```

### Blog posts list on home page

You have two options for the list of blog posts on the home page:

  - Show only the 5 most recent posts (default)

  ```
  customize:
    show_all_posts: false
    post_count: 5
  ```

  - Show all posts

  ```
  customize:
    show_all_posts: true
  ```

### Projects list

Create a projects file `source/_data/projects.json`.

  ```json
  [
      {
         "name":"Hexo",
         "url":"https://hexo.io/",
         "desc":"A fast, simple & powerful blog framework"
      },
      {
         "name":"Font Awesome",
         "url":"http://fontawesome.io/",
         "desc":"The iconic font and CSS toolkit"
      }
  ]
  ```

### Social media links

Cactus Light can automatically add links to your social media accounts. Therefore, update the theme's `_config.yml`:

  ```
  customize:
    social_links:
      github: your-github-url
      twitter: your-twitter-url
      NAME: your-NAME-url
  ```

where `NAME` is the name of a [Font Awesome icon](http://fontawesome.io/icons/#brand).

### RSS

Set the `rss` field in the theme's `_config.yml` to one of the following values:

1. `rss: false` will totally disable rss (default).
2. `rss: atom.xml` sets a specific feed link.
3. `rss:`leave empty to use the [hexo-generator-feed](https://github.com/hexojs/hexo-generator-feed) plugin.

### Analytics

Add you Google Analytics `tracking_id` to the theme's `_config.yml`.

  ```
  plugins:
      gooogle_analytics: 'UA-49627206-1'            # Format: UA-xxxxxx-xx
  ```

### Comments

First, create a site on Disqus: [https://disqus.com/admin/create/](http://disqus.com/admin/create/).

Next, update the theme's `_config.yml` file:

  ```
  plugins:
      disqus_shortname: SITENAME
  ```

where `SITENAME` is the name you gave your site on Disqus.

### Code Highlighting

Pick one of [the available colorschemes](https://github.com/gabithume/cactus-light/tree/master/source/css/_highlight) and add it to the theme's `_config.yml`:

  ```
  customize:
      highlight: COLORSCHEME_NAME
  ```

## License
MIT
