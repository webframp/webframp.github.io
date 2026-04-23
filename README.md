# webframp.com

Personal blog — ops culture and other hacker ramblings.

**Live site:** [webframp.com](https://webframp.com/)

## Stack

- [Hugo](https://gohugo.io/) (v0.155.3 extended) — static site generator
- [PaperMod](https://github.com/adityatelange/hugo-PaperMod) — theme (git submodule)
- [GitHub Pages](https://pages.github.com/) — hosting
- [GitHub Actions](.github/workflows/hugo.yml) — CI/CD

## Local development

Prerequisites: [Hugo extended](https://gohugo.io/installation/) (v0.155.3+)

```sh
git clone --recurse-submodules git@github.com:webframp/webframp.github.io.git
cd webframp.github.io
hugo server -D
```

The site is available at `http://localhost:1313/`. The `-D` flag includes draft posts.

## Writing a new post

```sh
hugo new posts/my-post-title.md
```

This creates a new post from the [default archetype](archetypes/default.md) with `draft: true`. Set `draft: false` when ready to publish.

## Project structure

```
archetypes/       # Templates for hugo new
assets/css/       # Custom CSS overrides (Glacial Indifference, Inter, JetBrains Mono)
content/          # Markdown content (posts, about, archives, search)
layouts/          # Layout overrides (shortcodes, partials)
static/fonts/     # Self-hosted font files
themes/PaperMod/  # Theme (git submodule — do not edit directly)
```

## Deployment

Pushes to `master` trigger the [GitHub Actions workflow](.github/workflows/hugo.yml), which builds the site and deploys to GitHub Pages. No manual deployment needed.

## License

Blog content is © Sean Escriva. Site configuration and templates are available under [MIT](https://opensource.org/licenses/MIT).
