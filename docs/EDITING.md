# Editing the blog

The blog is live at https://bulut19.github.io/stockout-aware-forecasting/ and is built from the `docs/` folder of the `blog` branch. Any change saved here shows up on the site within a minute or two.

## Change the text of a section

Each tab is one Markdown file in `docs/sections/`:

| Tab | File |
|---|---|
| Overview | `01-overview.md` |
| Data | `02-data.md` |
| Missed demand | `03-missed-demand.md` |
| Models | `04-models.md` |
| Evaluation | `05-evaluation.md` |
| Results | `06-results.md` |
| Business impact | `07-business-impact.md` |
| Conclusion | `08-conclusion.md` |
| References | `09-references.md` |

Easiest way: open the section on the site, click **edit this section on GitHub** at the bottom, make the change, then click **Commit changes** (commit directly to the `blog` branch).

Markdown basics: `## Heading`, `**bold**`, `*italic*`, `- bullet`, `[link text](https://...)`, tables with `|` columns.

## Add a picture

Upload the image to `docs/figures/` (on GitHub: open the folder, **Add file > Upload files**), then add this line where it should appear:

```
![Short description](figures/your_image.png)
```

## Add, rename or reorder tabs

Edit the `SECTIONS` list near the bottom of `docs/index.html`. Each entry has the tab's id (used in the link), its title and its file.

## Preview on your own computer

From the `docs/` folder run `python -m http.server`, then open http://localhost:8000. (Opening `index.html` directly does not work, because the page loads the section files.)
