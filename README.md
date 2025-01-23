# Kubeconform Helm

This repo contains tools that allow to use `kubeconform` to test [Helm
charts](https://helm.sh) in the form of [Helm
plugin](https://helm.sh/docs/topics/plugins/) and [`pre-commit`
hook](https://pre-commit.com/).

## Usage

### Helm plugin

The `kubeconform` [Helm plugin](https://helm.sh/docs/topics/plugins/) can be
installed using this command:

```shell
helm plugin install https://github.com/jtyr/kubeconform-helm
```

Once installed, the plugin can be used from any Helm chart directory:

```shell
# Enter the chart directory
cd charts/mychart
# Run kubeconform plugin
helm kubeconform .
```

The plugin uses `helm template` internally and passes its output to the
`kubeconform`. There is several `helm template` command line options supported
by the plugin that can be specified:

```shell
helm kubeconform --namespace myns .
```

There is also several `kubeconform` command line options supported by the plugin
that can be specified:

```shell
# Kubeconform options
helm kubeconform --verbose --summary .
```

It's also possible to create `.kubeconform` file in the Helm chart directory
that can contain default `kubeconform` settings:

```yaml
# Command line options that can be set multiple times can be defined as an array
schema-location:
  - default
  - https://raw.githubusercontent.com/datreeio/CRDs-catalog/main/{{.Group}}/{{.ResourceKind}}_{{.ResourceAPIVersion}}.json
# Command line options that can be specified without a value must have boolean
# value in the config file
summary: true
verbose: true
```

The full list of options for the
[plugin](https://github.com/jtyr/kubeconform-helm/blob/master/scripts/plugin_wrapper.py)
is as follows:

```text
$ ./scripts/plugin_wrapper.py --help
usage: plugin_wrapper.py [-h] [--cache] [--cache-dir DIR] [--config FILE]
                         [--values-dir DIR] [--values-pattern PATTERN] [-d] [--stdout]
                         [--errors-only] [--fail-fast] [--skip-refresh] [--verify]
                         [-f FILE] [-n NAME] [-r NAME] [--ignore-missing-schemas]
                         [--insecure-skip-tls-verify] [--kubernetes-version VERSION]
                         [--goroutines NUMBER] [--output {json,junit,tap,text}]
                         [--reject LIST] [--schema-location LOCATION] [--skip LIST]
                         [--strict] [--summary] [--verbose]
                         CHART

Wrapper to run kubeconform for a Helm chart.

options:
  -h, --help            show this help message and exit
  --cache               whether to use kubeconform cache
  --cache-dir DIR       path to the cache directory (default: ~/.cache/kubeconform)
  --config FILE         config file name (default: .kubeconform)
  --values-dir DIR      directory with optional values files for the tests (default:
                        tests/kubeconform)
  --values-pattern PATTERN
                        pattern to select the values files (default: *-values.yaml)
  -d                    debug output
  --stdout              log to stdout
  --errors-only         output only errors
  --fail-fast           fail on first error

helm build:
  Options passed to the 'helm build' command

  --skip-refresh        do not refresh the local repository cache
  --verify              verify the packages against signatures

helm template:
  Options passed to the 'helm template' command

  -f FILE, --values FILE
                        values YAML file or URL (can specified multiple)
  -n NAME, --namespace NAME
                        namespace
  -r NAME, --release NAME
                        release name
  CHART                 chart path (e.g. '.')

kubeconform:
  Options passsed to the 'kubeconform' command

  --ignore-missing-schemas
                        skip files with missing schemas instead of failing
  --insecure-skip-tls-verify
                        disable verification of the server's SSL certificate
  --kubernetes-version VERSION
                        version of Kubernetes to validate against, e.g. 1.18.0 (default:
                        master)
  --goroutines NUMBER   number of goroutines to run concurrently (default: 4)
  --output {json,junit,tap,text}
                        output format (default: text)
  --reject LIST         comma-separated list of kinds or GVKs to reject
  --schema-location LOCATION
                        override schemas location search path (can specified multiple)
  --skip LIST           comma-separated list of kinds or GVKs to ignore
  --strict              disallow additional properties not in schema or duplicated keys
  --summary             print a summary at the end (ignored for junit output)
  --verbose             print results for all resources (ignored for tap and junit output)
```

## Helm `pre-commit` hook

The `kubeconform` [`pre-commit` hook](https://pre-commit.com) can be added into the
`.pre-commit-config.yaml` file like this:

```yaml
repos:
  - repo: https://github.com/jtyr/kubeconform-helm
    rev: v0.1.17
    hooks:
      - id: kubeconform-helm
```

The hook uses `helm template` internally and passes its output to the
`kubeconform`. There is several `helm template` command line options supported
by the hook that can be specified:

```yaml
  - repo: https://github.com/jtyr/kubeconform-helm
    rev: v0.1.17
    hooks:
      - id: kubeconform-helm
        args:
          - --namespace=myns
          - --release=myrelease
```

There is also several `kubeconform` command line options supported by the hook
that can be specified:

```yaml
  - repo: https://github.com/jtyr/kubeconform-helm
    rev: v0.1.17
    hooks:
      - id: kubeconform-helm
        args:
          - --kubernetes-version=1.24.0
          - --verbose
          - --summary
```

The full list of options for the
[hook](https://github.com/jtyr/kubeconform-helm/blob/master/scripts/pre-commit.py)
is as follows:

```text
$ ./scripts/pre-commit.py --help
usage: pre-commit.py [-h] [--charts-path PATH] [--include-charts LIST]
                     [--exclude-charts LIST] [--path-sub-pattern PATTERN]
                     [--path-sub-separator SEP] [--cache] [--cache-dir DIR] [--config FILE]
                     [--values-dir DIR] [--values-pattern PATTERN] [-d] [--stdout]
                     [--errors-only] [--fail-fast] [--skip-refresh] [--verify] [-f FILE]
                     [-n NAME] [-r NAME] [--ignore-missing-schemas]
                     [--insecure-skip-tls-verify] [--kubernetes-version VERSION]
                     [--goroutines NUMBER] [--output {json,junit,tap,text}] [--reject LIST]
                     [--schema-location LOCATION] [--skip LIST] [--strict] [--summary]
                     [--verbose]
                     FILES [FILES ...]

Wrapper to run kubeconform for a Helm chart.

positional arguments:
  FILES                 files that have changed

options:
  -h, --help            show this help message and exit
  --charts-path PATH    path to the directory with charts, can be multiple (default:
                        charts)
  --include-charts LIST
                        comma-separated list of chart names to include in the testing
  --exclude-charts LIST
                        comma-separated list of chart names to exclude from the testing
  --path-sub-pattern PATTERN
                        substitution pattern to rewrite chart directory path for library
                        charts (e.g. '^charts/(commonlib),helper-charts/\1-test')
  --path-sub-separator SEP
                        separator used to split the path-sub-pattern (default: ,)
  --cache               whether to use kubeconform cache
  --cache-dir DIR       path to the cache directory (default: ~/.cache/kubeconform)
  --config FILE         config file name (default: .kubeconform)
  --values-dir DIR      directory with optional values files for the tests (default:
                        tests/kubeconform)
  --values-pattern PATTERN
                        pattern to select the values files (default: *-values.yaml)
  -d                    debug output
  --stdout              log to stdout
  --errors-only         output only errors
  --fail-fast           fail on first error

helm build:
  Options passed to the 'helm build' command

  --skip-refresh        do not refresh the local repository cache
  --verify              verify the packages against signatures

helm template:
  Options passed to the 'helm template' command

  -f FILE, --values FILE
                        values YAML file or URL (can specified multiple)
  -n NAME, --namespace NAME
                        namespace
  -r NAME, --release NAME
                        release name

kubeconform:
  Options passsed to the 'kubeconform' command

  --ignore-missing-schemas
                        skip files with missing schemas instead of failing
  --insecure-skip-tls-verify
                        disable verification of the server's SSL certificate
  --kubernetes-version VERSION
                        version of Kubernetes to validate against, e.g. 1.18.0 (default:
                        master)
  --goroutines NUMBER   number of goroutines to run concurrently (default: 4)
  --output {json,junit,tap,text}
                        output format (default: text)
  --reject LIST         comma-separated list of kinds or GVKs to reject
  --schema-location LOCATION
                        override schemas location search path (can specified multiple)
  --skip LIST           comma-separated list of kinds or GVKs to ignore
  --strict              disallow additional properties not in schema or duplicated keys
  --summary             print a summary at the end (ignored for junit output)
  --verbose             print results for all resources (ignored for tap and junit output)
```

## Contribution

This repo utilises [`pre-commit`](https://pre-commit.com/) hooks to lint code
changes. Make sure you install it before contributing to the repo.

### Installation

Following are the installation instructions for `pre-commit`. Further details
can be found [here](https://pre-commit.com/#installation).

#### Mac

```shell
brew install pre-commit
```

#### Ubuntu

```shell
pip install pre-commit
```

#### Arch Linux

```shell
pacman -S python-pre-commit
```

### Usage

`pre-commit` can run automatically on every commit. This requires to run the
following command once:

```shell
pre-commit install
```

Use the following command to run `pre-commit` manually for all files in the
repository:

```shell
pre-commit run --all-files
```

## Security Scan

### Manual Scan

To run security scans manually using Trivy, follow these steps:

1. Install Trivy by following the instructions on the [Trivy GitHub page](https://github.com/aquasecurity/trivy).
2. Run the security scan using the following command:

```shell
./scripts/security_scan.sh
```

### Pre-commit Hook

To configure the pre-commit hook for security scans, add the following entry to your `.pre-commit-config.yaml` file:

```yaml
repos:
  - repo: https://github.com/aquasecurity/trivy
    rev: v0.20.2
    hooks:
      - id: trivy-scan
        name: Trivy Security Scan
        entry: ./scripts/security_scan.sh
        language: system
        files: .*
```

## Debugging Pre-commit Hook

### Debugging Statements

To debug the pre-commit hook and print out its information, you can add debugging statements to the `scripts/pre-commit.py` file. This will help you understand which files are being passed to the hook and the values of important variables and arguments.

1. Open the `scripts/pre-commit.py` file in your preferred text editor.
2. Add the following debugging statements at the beginning of the `main` function to print the list of files being passed to the pre-commit hook:

```python
def main():
    # Parse args
    args = pw.parse_args(
        add_chart=False,
        add_files=True,
        add_path=True,
        add_incl_excl=True,
        add_path_sub=True,
    )

    # Debugging statement to print the list of files being passed to the pre-commit hook
    print("Files passed to pre-commit hook:", args["wrapper"].FILES)

    # We gonna change directory into the chart directory so we add it as local
    # path for helm dependency build and helm template
    args["helm_build"].append(".")
    args["helm_tmpl"].append(".")
```

3. Add the following debugging statements to print the values of important variables and arguments:

```python
    # Ger logger
    log = pw.get_logger(
        args["wrapper"].debug,
    )

    # Debugging statements to print the values of important variables and arguments
    print("Charts path:", args["wrapper"].charts_path)
    print("Include charts:", args["wrapper"].include_charts)
    print("Exclude charts:", args["wrapper"].exclude_charts)
    print("Path substitution pattern:", args["wrapper"].path_sub_pattern)
    print("Path substitution separator:", args["wrapper"].path_sub_separator)
```

4. Save the changes to the `scripts/pre-commit.py` file.

Now, when you run the pre-commit hook, it will print out the list of files being passed to the hook and the values of important variables and arguments, helping you debug any issues.

## License

MIT

## Author

Jiri Tyr
