# Resume

LaTeX source and automatic PDF publishing at
[resume.pdf](https://mariam-saeed.github.io/resume/resume.pdf).

```text
resume/
├── .github/workflows/deploy.yml  # Compile, validate PRs, deploy main
├── resume.tex                   # Main LaTeX document
├── .gitignore                   # Ignore PDFs and build artifacts
└── README.md
```

`resume.tex` contains the resume source. Keep its content and formatting there;
generated PDFs are never committed.

## Build locally

On Ubuntu 24.04, install the same package collections used in CI:

```sh
sudo apt-get update
sudo apt-get install --yes --no-install-recommends \
  texlive-xetex texlive-latex-base texlive-latex-recommended \
  texlive-latex-extra texlive-fonts-recommended lmodern \
  fonts-inter fontconfig
fc-cache -f
mkdir -p build
xelatex -no-shell-escape -interaction=nonstopmode -halt-on-error -file-line-error -output-directory=build resume.tex
xelatex -no-shell-escape -interaction=nonstopmode -halt-on-error -file-line-error -output-directory=build resume.tex
```

The result is `build/resume.pdf`. Required dependencies are:

| Dependency | Supplied by |
| --- | --- |
| XeLaTeX, XeTeX PDF backend (`xdvipdfmx`), TeX lookup tools | `texlive-xetex` and its dependencies |
| `article`, `color`, `fancyhdr`, `babel` (English), `array`, `fontspec`, `hyperref` and its XeTeX backend, standard math fonts | `texlive-latex-base`, `texlive-latex-recommended` and their dependencies |
| `fullpage`, `titlesec`, `enumitem` | `texlive-latex-extra` |
| Standard PostScript fonts and Zapf Dingbats metrics (`pzdr.tfm`) | `texlive-fonts-recommended` |
| Latin Modern fonts for LaTeX defaults | `lmodern` and its `fonts-lmodern` dependency |
| Inter (regular, bold, italic faces), font discovery/cache | `fonts-inter`, `fontconfig` |

CI installs these collections explicitly, including recommended fonts that
`--no-install-recommends` would otherwise omit. `hyperref`'s XeTeX link support
requires `pzdr` even when the document's text uses Inter. `marvosym` is unused
and remains removed. CI checks package files, standard font metrics, and Inter
availability before compiling. Ubuntu handles the system libraries, font maps,
and TeX formats through package dependencies and installation hooks. Git, Bash,
and standard file utilities are supplied by the Ubuntu runner. No bibliography,
external graphics, shell-escape tools, or additional language collections are
required by the current source.

The Ubuntu collections keep the existing TeX Live generation and avoid a full
TeX Live installation. Maintained full TeX Live Docker images were considered,
but add a large image download and a different toolchain without a benefit for
this source. If adopting one later, pin its digest and also pin the Inter source;
using `latest` alone would not make builds reproducible.

## GitHub Pages setup (once)

1. Use the repository `mariam-saeed/resume`, with `main` as the default branch.
   Use a public repository for GitHub Free, or a plan supporting private Pages.
2. In **Settings → Actions → General**, enable GitHub Actions and allow the
   official `actions/*` actions used by this workflow.
3. In **Settings → Pages → Build and deployment → Source**, select
   **GitHub Actions**. No publishing branch, custom domain, or secret is needed.
4. If the `github-pages` environment has deployment branch restrictions, allow
   `main`. Leave required reviewers unset for fully automatic deployments.

See GitHub's [custom Pages workflow documentation](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages).

## Update and publish

Edit `resume.tex`, then commit and push your changes to `main` yourself.
Pull requests targeting `main` compile for validation without uploading a Pages
artifact or deploying. Pushes to `main` compile twice, stop on any LaTeX error,
and publish only the generated PDF through the official `configure-pages`,
`upload-pages-artifact`, and `deploy-pages` actions. Check the **Actions** tab
for build errors and deployment status. A failed build leaves the hosted PDF
unchanged.

The PDF remains at `https://mariam-saeed.github.io/resume/resume.pdf`; there is
no index page at the site root. Generated PDFs and temporary files stay ignored.
The workflow grants read access to compilation and Pages/OIDC write access only
to deployment, and serializes runs per branch or PR to avoid overlapping deploys.

CI uses a fixed Ubuntu release, exact action release versions, and the source
commit timestamp for TeX's build date (`SOURCE_DATE_EPOCH` and
`FORCE_SOURCE_DATE`). Ubuntu package and runner security updates still apply;
this is a repeatable build setup, not a frozen, byte-for-byte toolchain snapshot.
