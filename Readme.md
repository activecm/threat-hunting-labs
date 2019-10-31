This is the source repo for https://activecm.github.io/threat-hunting-labs/

To update the website, just commit and push to this repository.

Resources:
- [Hugo](https://gohugo.io/) [Docs](https://gohugo.io/documentation/)
- [Learn Theme](https://learn.netlify.com/en/)
- [Github Workflow to Auto-Deploy](https://discourse.gohugo.io/t/deploy-hugo-project-to-github-pages-with-github-actions/20725)

Add new labs in the `content` directory. Just use the existing examples there for a template.

To customize the look and feel check out:
- `config.toml`
- `static/css/theme-mine.css`
- `static/css/custom.css`

To override something that exists in the default Learn theme, do not touch the files in `themes/`. Instead add the same file create a new file in the top level (e.g. under `layouts` or `static`) that has the same name as the one in the theme. Hugo will give this file priority over the one in the theme. See [here](https://learn.netlify.com/en/basics/style-customization/) for more information.
