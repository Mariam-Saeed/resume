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

`resume.tex` is currently a placeholder that intentionally fails compilation.
Replace the entire file with your existing source, unchanged, before the first
build. Include any source assets or custom `.cls`/`.sty` files it needs.

## Build locally

On Ubuntu 24.04, install the same minimal packages used in CI:

```sh
sudo apt-get update
sudo apt-get install --yes --no-install-recommends texlive-xetex fonts-inter fontconfig
fc-cache -f
mkdir -p build
xelatex -no-shell-escape -interaction=nonstopmode -halt-on-error -file-line-error -output-directory=build resume.tex
xelatex -no-shell-escape -interaction=nonstopmode -halt-on-error -file-line-error -output-directory=build resume.tex
```

The result is `build/resume.pdf`. XeLaTeX supports `fontspec` and the installed
Inter font. `texlive-xetex` also brings the standard LaTeX, recommended, extra,
and graphics packages through its dependencies. If your source needs additional
packages, add only their required Ubuntu packages to CI and the local command.
The exact additional requirements can be checked once the source is supplied.

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
