# 🐍 Python Package Guide

> A reference guide covering the essentials of Python packaging — including package structure, `pyproject.toml` configuration, and a comparison of popular build tools like PDM, Flit, Hatch, and Poetry.

---

## 📑 Table of Contents

1. [Need of a Python Package](#1-need-of-a-python-package)
2. [Elements of a Python Package](#2-elements-of-a-python-package)
3. [Package Directory Structure](#3-package-directory-structure)
4. [pyproject.toml File](#4-pyprojecttoml-file)
   - [4.1 Optional vs Required Fields](#41-optional-vs-required-fields)
   - [4.2 Build-System Table](#42-build-system-table)
5. [Build Tool Comparison](#5-build-tool-comparison)
   - [5.1 PDM Features](#51-pdm-features)
   - [5.2 Flit Features](#52-flit-features)
   - [5.3 Hatch Features](#53-hatch-features)
   - [5.4 Poetry Features](#54-poetry-features)
6. [project.urls Table](#6-projecturls-table)

---

## 1. Need of a Python Package

- Use your code across different projects
- Share code

---

## 2. Elements of a Python Package

- **Code**
- **Documentation** — Installation instructions and tutorials for users
- **Tests** — To make sure your code works as it should
- **License** — Open source license that allows others to use your package
- **Infrastructure** — Automates updates, runs test suites and CI/CD

---

## 3. Package Directory Structure

Example of a typical Python package directory structure:

```
pyospackage/               # Your project directory
 └─ pyproject.toml         # Define project's metadata, dependencies
 └─ src/                   # The source (src) directory ensures your tests always run on the installed version of your code
    └── pyospackage/       # Package directory where code lives
        ├── __init__.py    # Tells Python that a directory should be treated as a Python package
        ├── add_numbers.py # Code
        └── # Add any other .py modules that you want here
```

---

## 4. pyproject.toml File

Stores metadata that provides instructions to various tools interacting with it.

**TOML format basics:**
- Consists of **tables** and **variables**
- Tables are sections of information denoted by square braces
- Tables contain variables defined by a variable name and an `=`

---

### 4.1 Optional vs Required Fields

**Required fields in `[project]` table:**
- `name`
- `version`

**Optional fields in `[project]` table:**

| Field | Description |
| --- | --- |
| `description` | One-line description of the package |
| `readme` | Path to the readme file |
| `requires-python` | Tell the installer whether you use Python 2.x or 3.x |
| `license` | License you are using |
| `authors` | Package authors |
| `maintainers` | Package maintainers |
| `project.dependencies` | Optional because not all packages require dependencies |
| `project.optional` | Optional or feature dependencies; installed if someone runs `python -m pip install projectname[feature]` |
| `dependency-groups` | Organize package and tools that a developer would need to work on the package |
| `keywords` | Keywords that will appear on your PyPI landing page |
| `classifiers` | Important for the landing page of your package in PyPI |

---

### 4.2 Build-System Table

```toml
[build-system]
requires = []
build-backend = []  # What build backend to use to build the package
```

**Available build backends and tools:**

| **Tool** | **Type** | **Best For** |
| --- | --- | --- |
| setuptools | Backend | Complex / legacy builds |
| flit | Backend | Simple pure Python libs |
| hatch | Backend + env mgr | Modern workflows, automation |
| pdm | Manager + backend | PEP 582, lightweight dev |
| poetry | Full manager | Teams, dependency control |
| build | Frontend | Standard build interface |
| twine | Upload tool | Publishing to PyPI |

---

## 5. Build Tool Comparison

### 5.1 PDM Features

| **Feature** | **PDM** | **Notes** |
| --- | --- | --- |
| Use Other Build Backends | ✅ | When you setup PDM it allows you to select one of several build back ends including: PDM-core, flit-core and hatchling. PDM also can work with Meson-Python which supports more complex Python builds. |
| Dependency specifications | ✅ | PDM has flexible support for managing dependencies. PDM defaults to using an open bound (e.g. `requests >=1.2`) approach to dependencies. However you can customize how you want to add dependencies in case you prefer another approach such as that of Poetry which uses an upper bound limit. |
| Environment lock files | ✅ | PDM and Poetry are currently the only tools that create environment lock files. Lock files are often most useful to developers creating web apps where locking the environment is critical for consistent user experience. For community-used packages, you will likely never want to use a lock file. |
| Environment management | ✅ | PDM provides environment management support. It supports Python virtual environments, conda and a local `__pypackages__` environment which is a newer option in the Python ecosystem. No extensions are needed for this support. |
| Select your environment type on install | ✅ | When you run `pdm init`, PDM will discover environments that are already on your system and allow you to select one to use for your project. |
| Publish to PyPI | ✅ | PDM supports publishing to both test PyPI and PyPI |
| Version Control based versioning | ✅ | PDM has a `setuptools_scm` like tool built into it which allows you to use dynamic versioning that rely on git tags. |
| Version bumping | ✅ | PDM supports bumping the version of your package using standard semantic version terms `patch`, `minor`, `major` |
| Follows current packaging standards | ✅ | PDM supports current packaging standards for adding metadata to the `pyproject.toml` file. |
| Install your package in editable mode | ✅ | PDM supports installing your package in editable mode. |
| Build your sdist and wheel distributions | ✅ | Similar to all of the other tools, PDM builds your package's sdist and wheel files for you. |

---

### 5.2 Flit Features

| **Feature** | **Flit** | **Notes** |
| --- | --- | --- |
| Publish to PyPI and test PyPI | ✅ | Flit supports publishing to both test PyPI and PyPI |
| Helps you add metadata to your `pyproject.toml` file | ✅ | Flit does support adding metadata to your `pyproject.toml` file following modern packaging standards. |
| Follows current packaging standards | ✅ | Flit supports current packaging standards for adding metadata to the `pyproject.toml` file. |
| Install your package in editable mode | ✅ | Flit supports installing your package in editable mode. |
| Build your sdist and wheel distributions | ✅ | Flit can be used to build your package's sdist and wheel distributions. |

---

### 5.3 Hatch Features

| **Feature** | **Hatch** | **Notes** |
| --- | --- | --- |
| Use Other Build Backends | ✅ | Hatch is used with the backend Hatchling by default, but allows you to use another backend by switching the declaration in `pyproject.toml`. |
| Dependency management | ✖ | Currently you have to add dependencies manually with Hatch. However a feature to support dependency management may be added in a future release. |
| Environment Management | ✅ | Hatch supports Python virtual environments. If you wish to use other types of environments such as Conda, you will need to install a plugin such as `hatch-conda` for conda support. |
| Publish to PyPI and test PyPI | ✅ | Hatch supports publishing to both test PyPI and PyPI |
| Version Control based versioning | ✅ | Hatch offers `hatch_vcs` which is a plugin that uses `setuptools_scm` to support versioning using git tags. The workflow with `hatch_vcs` is the same as that with `setuptools_scm`. |
| Version bumping | ✅ | Hatch supports bumping the version of your package using standard semantic version terms `patch`, `minor`, `major` |
| Follows current packaging standards | ✅ | Hatch supports current packaging standards for adding metadata to the `pyproject.toml` file. |
| Install your package in editable mode | ✅ | Hatch will install your package into any of its environments by default in editable mode. You can install your package in editable mode manually using `python -m pip install -e .` |
| Build your sdist and wheel distributions | ✅ | Hatch will build the sdist and wheel distributions |
| ✨ Matrix environment creation to support testing across Python versions ✨ | ✅ | The matrix environment creation is a feature that is unique to Hatch in the packaging ecosystem. This feature is useful if you wish to test your package locally across Python versions (instead of using a tool such as tox). |
| ✨ Nox / MAKEFILE like functionality ✨ | ✅ | This feature is also unique to Hatch. This functionality allows you to create workflows in the `pyproject.toml` configuration to do things like serve docs locally and clean your package build directory. This means you may have one less tool in your build workflow. |
| ✨ A flexible build backend: **hatchling** ✨ | ✅ | The hatchling build backend offered by the maintainer of Hatch allows developers to easily build plugins to support custom build steps when packaging. |

---

### 5.4 Poetry Features

| **Feature** | **Poetry** | **Notes** |
| --- | --- | --- |
| Add dependencies to your `pyproject.toml` file | ✅ | Poetry helps you add dependencies to your `pyproject.toml` metadata. |
| Dependency specification | ✅ | Poetry allows you to be specific about the version of dependencies that you add to your package's `pyproject.toml` file. However, its default upper bound approach can be problematic for some packages (override the default setting when adding dependencies is suggested). |
| Environment management | ✅ | Poetry allows you to either use its built-in environment or you can select the environment type that you want to use for managing your package. |
| Lock files | ✅ | Poetry creates a `poetry.lock` file that you can use if you need a lock file for your build. |
| Publish to PyPI and test PyPI | ✅ | Poetry supports publishing to both test PyPI and PyPI |
| Version Control based versioning | ✅ | The plugin Poetry dynamic versioning supports versioning using git tags with Poetry. |
| Version bumping | ✅ | Poetry supports bumping the version of your package using standard semantic version terms `patch`, `minor`, `major` |
| Follows current packaging standards | ✅ | Since version 2.0, Poetry supports most current project metadata standards. However, not all standards are supported, and it also supports the legacy Poetry format. |
| Install your package in editable mode | ✅ | Poetry supports installing your package in editable mode. |
| Build your sdist and wheel distributions | ✅ | Poetry will build your sdist and wheel distributions using `poetry build` |

---

## 6. project.urls Table

Used to provide links related to the project:

- **Homepage**
- **Bug Reports**
- **Source**
