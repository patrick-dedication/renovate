---
title: Python Package Manager Support
description: Python/pip dependencies support in Renovate
---

# Python package manager support

Renovate supports several Python package managers, including `pip`, `pipenv`, `poetry`, etc.
See [all supported managers](./modules/manager/index.md).

## Versioning support

We've written a JavaScript version of the [PEP440 specification](https://www.python.org/dev/peps/pep-0440/) so we can use it in Renovate bot.
You can find this project in our [`renovatebot/pep440` repository](https://github.com/renovatebot/pep440).

Our PEP440 implementation supports pinned versions and ranges.
Legacy versions with the `===` prefix are ignored.

## How it works

1. Renovate searches through each repository for package files
1. Existing dependencies are extracted from the package files
1. Renovate searches for the latest version on [PyPI](https://pypi.org/) to decide if there are upgrades
1. If the source package includes a GitHub URL as its source, and has a "changelog" file _or_ uses GitHub releases, a Release Note will be embedded in the generated PR

## Package name matching

Your `matchPackageName` or `matchPackagePattern` rules will be matching against normalized names.
So if you have specified package `some.package` or `ANOTHER_DEP` in your package files (`requirements.txt`, `pyproject.toml`), they will be treated as `some-package` and `another-dep` respectively.
Not only they will be case insensitive but will replace any amount `._-` to a single `-`.
[Consult Python packaging documentation for the specification](https://packaging.python.org/en/latest/specifications/name-normalization/).

## Alternate registries

By default Renovate checks for upgrades on the `pypi.org` registry.

If you want, you can set alternative index URLs.
There are three ways to do this:

- index-url in `requirements.txt`
- sources in `Pipfile`
- sources in `pyproject.toml`
- set URL in Renovate configuration

### index-url in `requirements.txt`

```title="Setting index URL in first line of requirements.txt"
--index-url http://example.com/private-pypi/
some-package==0.3.1
some-other-package==1.0.0
```

### Sources in `Pipfile`

Renovate finds and uses any custom sources in your `Pipfile`.

### Sources in `pyproject.toml`

Renovate detects any custom-configured sources in `pyproject.toml` and uses them.

### Specify URL in configuration

Create a `packageRules` entry with `matchDatasources=pypi` and a `registryUrls` array.
Fill the array with alternate index URL(s).

```json
{
  "packageRules": [
    {
      "matchDatasources": ["pypi"],
      "registryUrls": ["http://example.com/private-pypi/"]
    }
  ]
}
```

<!-- prettier-ignore -->
!!! tip
    If a `requirements.txt` file has an index-url then Renovate follows that link, instead of following any link set in the `registryUrls` array.
    To override the URL found in `requirements.txt`, you must create a custom `packageRules` setting.
    This is because `packageRules` are applied _after_ package file extraction.

## Disabling Python support

```json title="Disabling all managers where language is set to Python"
{
  "python": {
    "enabled": false
  }
}
```

Alternatively, you can use `enabledManagers` to tell Renovate what package managers it can use.

```json title="Only use Renovate's npm package manager"
{
  "enabledManagers": ["npm"]
}
```
