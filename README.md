# PyLint with badge - GitHub Action

**This repository is a fork of `Silleellie/pylint-github-action` and still in beta**

GitHub action that lets you *easily* lint **one** or **multiple** packages (or python files) of your project and adds a **dynamic badge**
to your `README.md` that lets you display the obtained score!

Each time the action is run, packages (or python files) specified will be linted and a badge in the `README.md` is updated dynamically
following one of the below rules:

|              Range PyLint score               |                                                           Badge                                                           |
|:---------------------------------------------:|:-------------------------------------------------------------------------------------------------------------------------:|
|      **Bad score**: *PyLint score* $< 5$      |  ![PyLint-red](https://github.com/Silleellie/pylint-github-action/assets/26851363/27950618-f456-4fe6-a195-43e2d3f9015e)   |
|  **Ok score**: $5 \le$ *PyLint score* $< 8$   | ![PyLint-orange](https://github.com/Silleellie/pylint-github-action/assets/26851363/6718c99e-1cc9-4e85-84db-51a1f56d2051) |
| **Good score**: $8 \le$ *PyLint score* $< 10$ | ![PyLint-yellow](https://github.com/Silleellie/pylint-github-action/assets/26851363/6afe3441-193f-4cc7-8b64-a57110216c34) |
|   **Perfect score**: *PyLint score* $= 10$    |   ![PyLint-10](https://github.com/Silleellie/pylint-github-action/assets/26851363/64fb3a03-c3b9-48ad-8f66-cea41a0ccaf8)   |


**NEW:** You can now fully customize the badge color of each of the above ranges! Check [usage](#usage) 
and [scenario](#scenario) sections for more!


The action can be triggered by a **`Pull request`**, a **`Push`** or manually with **`workflow_dispatch`**. 

* **IMPORTANT!** Follow the ['Preliminary steps' section](#preliminary-steps) in order to allow the bot to update your 
README.md with the pylint badge!


A quick example on how you would typically use this *action* (more examples in [scenario section](#scenario))

```yaml
- name: PyLint Scanning
  id: pylint-scanning
  uses: YaoYinYing/pylint-github-action@v3.0
  with:
    lint-path: src  # lint src package
    python-version: ${{ env.lint-python-version }}  # python version which will lint the package
    badge-text: pylint score
    badge-file-name: pylint_scan
    conda-env-name: my-package
          
```

And after running the action, the *GitHub action* will update your PyLint badge file to artifacts named `upload-my-awesome-badge`

## Preliminary steps

To use this action you should perform two simple **first-time-only** operations:

1. In order to have a dynamic updated badge, before using for the first time this action, you should have a third party storage access (R2, S3, etc.) with publicly readable bucket.
2. Add a section to your GA yml to download updated badge from artifact and upload it to R2:
```yaml
- jobs:
  DevTests: # # Previous run with linting
    ...
  
  UpdateLinting:
    runs-on: 'ubuntu-latest'
    needs: DevTests # Previous run with linting as trigger
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
      - name: PyLint Badge Fetching
        id: pylint-badge-fetch
        uses: actions/download-artifact@v4
        with:
          name: upload-my-awesome-badge
          path: badge_dir_with_uniq_name
      - name: Display structure of downloaded files
        run: ls -R badge_dir_with_uniq_name
      - name: Upload to R2
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          command: r2 object put  ${{ secrets.R2_BUCKET }}/badge_dir_with_uniq_name/my-code/pylint/pylint_scan.svg --file badge_dir_with_uniq_name/pylint_scan.svg --content-type image/svg+xml
```
After lint running, a new badge will be upload to R2 bucket `${{ secrets.R2_BUCKET }}`. 

3. Now you can add a markdown record to your README.md for this pylint badge: 
```text
[![pylint](https://R2-bucket-url/badge_dir_with_uniq_name/my-code/pylint/pylint_scan.svg)](https://github.com/YaoYinYing/pylint-github-action)
```
4. Done!


## Full Usage Explained

```yaml
- uses: YaoYinYing/pylint-github-action@v3.0
  with:
    # The path, relative to the root of the repo, of the package(s) or pyton file(s) to lint
    lint-path: src
    
    # Python version which will install all dependencies and lint package(s)
    python-version: 3.11

    # Conda environment name for the package.
    conda-env-name: base

    # The path, relative to the root of the repo, of the requirements to install. If no such file here, leave it as default.
    requirements-path: requirements.txt

    # The path, relative to the root of the repo, of the README.md to update with the pylint badge
    readme-path:
    description: "README.md"

    # Text to display in the badge
    badge-text: PyLint

    # Color of the badge for pylint scores < 5. Hex, rgb, rgba, hsl, hsla and css named colors can all be used
    color-bad-score: red

    # Color of the badge for pylint scores in range [5,8). Hex, rgb, rgba, hsl, hsla and css named colors can all be used
    color-ok-score: orange

    # Color of the badge for pylint scores in range [8,10). Hex, rgb, rgba, hsl, hsla and css named colors can all be used
    color-good-score: yellow

    # Color of the badge for pylint scores == 10. Hex, rgb, rgba, hsl, hsla and css named colors can all be used
    color-perfect-score: brightgreen

    # The name of the artifact name for the badge file to upload.
    badge-artifact-name: upload-my-awesome-badge

    # The basename of badge file. 
    badge-file-name: badge
```

## Scenario

* [Single package to lint](#single-package-to-lint)
* [Single python file to lint](#single-python-file-to-lint)
* [Multiple packages to lint](#multiple-packages-to-lint)
* [Multiple python files to lint](#multiple-python-files-to-lint)
* [Mix packages and python files to lint](#mix-packages-and-python-files-to-lint)
* [Different path for requirements file](#different-path-for-requirements-file)
* [Different path for README.md file](#different-path-for-readmemd-file)
* [Change badge text](#change-badge-text)
* [Change badge color with css named color](#change-badge-color-with-css-named-color)
* [Change badge color with hex code](#change-badge-color-with-hex-code)

### Single package to lint

```yaml
- uses: YaoYinYing/pylint-github-action@v3.0
  with:
    lint-path: src
    python-version: 3.11
```

### Single python file to lint

```yaml
- uses: YaoYinYing/pylint-github-action@v3.0
  with:
    lint-path: main.py
    python-version: 3.11
```

### Multiple packages to lint

```yaml
- uses: YaoYinYing/pylint-github-action@v3.0
  with:
    lint-path: |
      src
      app
      other_src/inner_src
    python-version: 3.11
```

### Multiple python files to lint

```yaml
- uses: YaoYinYing/pylint-github-action@v3.0
  with:
    lint-path: |
      file1.py
      file2.py
      other_src/file3.py
    python-version: 3.11
```

### Mix packages and python files to lint

```yaml
- uses: YaoYinYing/pylint-github-action@v3.0
  with:
    lint-path: |
      src
      app
      main.py
    python-version: 3.11
```

### Different path for requirements file

```yaml
- uses: YaoYinYing/pylint-github-action@v3.0
  with:
    lint-path: src
    python-version: 3.11
    requirements-path: requirements/requirements-dev.txt
```

### Different path for README.md file

```yaml
- uses: YaoYinYing/pylint-github-action@v3.0
  with:
    lint-path: src
    python-version: 3.11
    readme-path: models/README.md
```

### Change badge text

```yaml
- uses: YaoYinYing/pylint-github-action@v3.0
  with:
    lint-path: src
    python-version: 3.11
    badge-text: alternative text
```

### Change badge color with css named color

In this case we are extending what we consider a perfect score: all scores in range $[8, 10]$ are considered
good enough and will have same color (*brightgreen*)

```yaml
- uses: YaoYinYing/pylint-github-action@v3.0
  with:
    lint-path: src
    python-version: 3.11
    color-good-score: brightgreen
    color-perfect-score: brightgreen
```

### Change badge color with hex code

In this example we are changing the color for the *bad score range* ($[0,5)$) to purple (hex code: *800080*)

```yaml
- uses: YaoYinYing/pylint-github-action@v3.0
  with:
    lint-path: src
    python-version: 3.11
    color-bad-score: 800080
```

## Credits

This is a composite GitHub action which uses the following godly working actions:

* [Silleellie/pylint-github-action](https://github.com/Silleellie/pylint-github-action)
* [actions/checkout](https://github.com/actions/checkout)
* [actions/upload-artifact](https://github.com/actions/upload-artifact)
* [actions/download-artifact](https://github.com/actions/download-artifact)
* [actions/setup-python](https://github.com/actions/setup-python)
* [cloudflare/wrangler-action](https://github.com/cloudflare/wrangler-action)


Massive thanks to [shields.io](https://shields.io/), which is used to create the badge!
