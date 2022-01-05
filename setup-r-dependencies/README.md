# setup-r-dependencies

[![RStudio community](https://img.shields.io/badge/community-github--actions-blue?style=social&logo=rstudio&logoColor=75AADB)](https://community.rstudio.com/new-topic?category=Package%20development&tags=github-actions)

This action install dependencies for the current R environment based on the DESCRIPTION file in the repository by:

- Installing [pak](https://pak.r-lib.org/)
- Setting up a dependency cache using [actions/cache](https://github.com/actions/cache).
- Installing system dependencies if needed using [rstudio/r-system-requirements](https://github.com/rstudio/r-system-requirements).
- Printing the installed session info using [sessioninfo](https://github.com/r-lib/sessioninfo).

# Usage

Inputs available

- `cache-version` - default `1`. If you need to invalidate the existing
  cache pass any other number and a new cache will be used.
- `dependencies` - default `'"all"'`. Types of dependencies to install. By
  default all direct dependencies of the current package are installed, and
  hard dependencies of these direct dependencies. See also the `needs` and
  `extra-packages` parameters. This parameter must be a valid R expression
  in single quotes and it is passed to the `dependencies` argument of
  `pak::lockfile_create().`
- `extra-packages` - One or more extra package references to install.
  Separate each reference by newlines or commas for more than one package.
- `needs` - `Config/Needs` fields to install from the DESCRIPTION, the
  `Config/Needs/` prefix will be automatically included.
- `pak-version`: Which pak version to use. Possible values are
  `stable`, `rc` and `devel`. Currently it defaults to `devel`, because some
  of the new features of the `v2` release depend on that, but the default
  will change to `stable` soon. You can override the default with this
  parameter.
- `working-directory` - default `'.'`. If the DESCRIPTION file is not in the
  root directory of your repository.

Basic:
```yaml
steps:
- uses: actions/checkout@v2
- uses: r-lib/actions/setup-r@v2
- uses: r-lib/actions/setup-r-dependencies@v2
  with:
    cache-version: 2
    extra-packages: |<
      any::ggplot2
      any::rcmdcheck
    needs: |
      website
      coverage
```

## Supported R versions and operating systems

`setup-r-dependencies` uses static builds of [pak](https://pak.r-lib.org/)
to resolve, download and install the dependencies. There are typically
recent static pak builds available for:

- x86_64 Linux, for the last 5 R releases and R-devel (currently this is
  R 3.4.x through R 4.1.x and R-devel).
- x86_64 macOS, for the last 5 R releases and R-devel.
- Windows (x86_64 and i386), for the last 5 R releases and R-devel.

There are typically less recent builds for

- arm64 macOS, from R 4.1.x, but at most the last 5 R releases, and R devel.

See https://github.com/r-lib/pak#installation for the most accurate
information.

If your platform does not have a static pak build, e.g. you are on
s390x Linux, then you cannot use the `setup-r-dependencies` action currently.

## Extra packages and the `any::` prefix

In the example above the `any::` prefix for ggplot2 and rcmdcheck tells pak
to install these packages from CRAN, unless the local package or one of
its dependencies request it from somewhere else. E.g. if the checked package
required the development version of ggplot2 from
https://github.com/tidyverse/ggplot2 then pak will install it from there.

## Ignoring optional dependencies that need a newer R version

When you check a package on an earlier R version, it is not uncommon that
some optional (soft) dependencies of your package are not available on that
R version because they need a newer one.

You can use the `extra-packages` parameter and pak's `package=?ignore-before-r`
syntax to express this.

Here is an example. The butcher package depends on survival, which needs
R 3.5.0 now, and pak's dependency resolution fails on R 3.4.x:

```
Error: Cannot install packages:
* local::.: Can't install dependency survival
* survival: Needs R 3.5.0
```

To tell pak to ignore survival on R versions older than R 3.5.0, you can
write this in the butcher workflow file:

```yaml
steps:
- uses: actions/checkout@v2
- uses: r-lib/actions/setup-r@v2
- uses: r-lib/actions/setup-r-dependencies@v2
  with:
    extra-packages: any::rcmdcheck, survival=?ignore-before-r=3.5.0
    needs: check
```

Of course you'll also have to make sure that the test cases and examples
in butcher are prepared for survival not being available. For running
`R CMD check` you'll probably also need to set the `_R_CHECK_FORCE_SUGGESTS_`
environment variable to `false`, otherwise the check fails if suggested
packages are not available:

```yaml
- uses: r-lib/actions/check-r-package@v2
  env:
    _R_CHECK_FORCE_SUGGESTS_: false
```

## Installing the local package

Sometimes you need to install the R package in the repository, e.g.
the `pkgdown.yaml` example workflow does this. You can specify the local
package as `local::.` to pak:

```yaml
steps:
- uses: actions/checkout@v2
- uses: r-lib/actions/setup-r@v2
- uses: r-lib/actions/setup-r-dependencies@v2
  with:
    extra-packages: any::pkgdown, local::.
    needs: website
```

Other packages from the repository can be installed similarly, e.g.
to install an embedded test package you can write:

```yaml
...
- uses: r-lib/actions/setup-r-dependencies@v2
  with:
    extra-packages: any::pkgdown, local::., local::./tests/testthat/testpkg
...
```

## System dependencies

pak automatically install the system dependencies of all installed R
packages on the following Linux Distributions:

* CentOS,
* OpenSUSE,
* RedHat,
* SLE and
* Ubuntu.

System dependencies are **not** installed on other operating systems and
other Linux distributions currently, and you need to install them manually,
_before_ using the `r-lib/setup-r-dependencies` action.

On macOS you can usually use `brew`, here is an example step in a workflow:

```yaml
      - name: Install macOS system dependencies
        if: runner.os == 'macOS'
        run: |
          brew install imagemagick@6
          brew install libgit2
```

On Windows you can usually use `pacman` that is included in Rtools4, or
`choco` to install external software:

```yaml
      - name: Install most Windows system dependencies
        if: runner.os == 'Windows'
        run: |
          pacman -Syu mingw-w64-x86_64-make --noconfirm
```

```yaml
      - name: Install databases (windows)
        if: runner.os == 'Windows'
        shell: bash
        run: |
          choco install mariadb
```

# License

The scripts and documentation in this project are released under the [MIT License](LICENSE)

# Contributions

Contributions are welcome!