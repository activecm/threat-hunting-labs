

Resources:
- [Hugo](https://gohugo.io/) [Docs](https://gohugo.io/documentation/)
- [Learn Theme](https://learn.netlify.com/en/)

Add new labs in the `content` directory. Just use the existing examples there for a template.

To customize the look and feel check out:
- `config.toml`
- `static/css/theme-mine.css`
- `static/css/custom.css`

To override something that exists in the default Learn theme, do not touch the files in `themes/`. Instead add the same file create a new file in the top level (e.g. under `layouts` or `static`) that has the same name as the one in the theme. Hugo will give this file priority over the one in the theme. See [here](https://learn.netlify.com/en/basics/style-customization/) for more information.
