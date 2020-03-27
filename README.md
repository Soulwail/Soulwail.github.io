## Welcome to GitHub Pages

## jekyll serve

If you indeed go for setting up the `baseurl`, you can view your site locally via `localhost: 4000` by adding this flag when serving Jekyll: `--baseurl ""`. So, `jekyll serve --watch --baseurl ""`. This means like "Jekyll, ignore ths baserurl set in the config".

macOS

```javascript
jekyll serve --watch --baseurl ""
```

windows

```javascript
jekyll serve --watch --baseurl=""
```

## Curly braces cannot be used in markdown

Use `raw` block, such as `{% raw %} {{ temp }} {% endraw %}`

[https://hexo.io/docs/troubleshooting.html#Escape_Contents](https://hexo.io/docs/troubleshooting.html#Escape_Contents)
