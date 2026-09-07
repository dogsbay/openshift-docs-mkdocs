# openshift-docs-mkdocs

OpenShift's documentation as an **MkDocs (Material)** site, generated from
the Dogsbay MD in [openshift-docs-markdown](https://github.com/dogsbay/openshift-docs-markdown)
and published to GitHub Pages.

This branch (`main`) holds only the workflow. **The exported project lives
on version branches**, named after the source branch:

| Branch | Source | Published |
|---|---|---|
| [`enterprise-4.22`](../../tree/enterprise-4.22) | [openshift-docs-markdown@enterprise-4.22](https://github.com/dogsbay/openshift-docs-markdown/tree/enterprise-4.22) | https://dogsbay.github.io/openshift-docs-mkdocs/ |

## What this shows

One canonical format, three static-site generators. The same Dogsbay MD is
published as

- an Astro site — [openshift-docs-markdown](https://github.com/dogsbay/openshift-docs-markdown) (`site/`)
- **this MkDocs site**
- a Docusaurus site — [openshift-docs-docusaurus](https://github.com/dogsbay/openshift-docs-docusaurus)

and because all three are `dogsbay site build --to <format>` — one pipeline
pass (attributes, conditionals and includes resolved, `routablesFrom: nav`
applied), then a serializer — they are built from an identical page set and
can be compared page for page.

## How it runs

One workflow, **Build MkDocs site**, run on dispatch and on Mondays 07:00 UTC
(two hours after the markdown repo's sync, though scheduled runs can slip). It takes the source branch as an
input and:

1. sparse-checks-out `markdown/` + `dogsbay.config.yml` from the source branch;
2. runs the export:

   ```
   dogsbay site build source --to mkdocs --out out \
     --site-url https://dogsbay.github.io/openshift-docs-mkdocs
   ```

3. builds it with `mkdocs-material` (about 16 minutes for 1,800 pages —
   MkDocs is single-threaded), refusing a run under 1,000 pages;
4. commits `mkdocs.yml` + `docs/` to the version branch (the built HTML is
   not committed — Pages deploys it from the artefact);
5. deploys to GitHub Pages and frees the artefact.

The build refuses to deploy a site over 950 MB — GitHub Pages' limit is
1 GB. Pages itself is enabled once at repo creation with source "GitHub
Actions"; the workflow cannot enable it with the default token.

`--site-url` matters on a project Pages site: MkDocs writes every absolute
link from `site_url`, and without the `/openshift-docs-mkdocs` path they
all 404.

The build is not `--strict`. MkDocs warns on every anchor the source
itself gets wrong, and the AsciiDoc has hundreds; the warning count is
reported in the log rather than made a gate.

## Fidelity

Measured on `enterprise-4.22` against AsciiBinder's own HTML for a complex
page (bare-metal UPI network customizations): 166/166 code blocks, 117/117
admonitions, ~98.7% token retention. What remains is traced to the source:
`== Next steps` demoted to a related-links fold, `_`-prefixed ids, table
numbering. Details in the Dogsbay repo's `docs-dev/format-mkdocs-export.md`.
