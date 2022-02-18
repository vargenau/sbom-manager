# SBOM Manager

[![DepShield Badge](https://depshield.sonatype.org/badges/owner/repository/depshield.svg)](https://depshield.github.io)

The SBOM Manager is a free, open source tool to help you manage a collection of SBOMs(Software Bill of Materials) in a number of formats including
[SPDX](https://www.spdx.org) and [CycloneDX] (https://www.cyclonedx.org).

The tool has two main modes of operation:

1. A repository which maintains the set of components which have been included as part of a release or build of a software product.
2. Tools for quering the inclusion of specific products in a project development to answer some commmon use cases.

It is intended to be used as part of a continuous integration system to enable regular records of SBOMs to be maintained 
and also to support security audit needs to determine if a particular component (and version) has been used.

## Installation

To install, just clone the repo and install dependencies using the following command:

`pip install -U -r requirements.txt`

The tool requires Python 3 (3.7+). It is recommended to use a virtual python environment especially 
if you are using different versions of python. `virtualenv` is a tool for setting up virtual python environments which
allows you to have all the dependencies for the tool set up in a single environment, or have different environments set
up for testing using different versions of Python.

## Usage

```
python sbom.py [-h] [-a ADD_FILE] [-t {spdx,cyclonedx,csv,dir}]
                [-l {all,sbom,module}] [-m MODULE] [-d DESCRIPTION]
                [-p PROJECT] [-s] [-q]
                [-L {debug,info,warning,error,critical}] [-o OUTPUT_FILE]
                [-f {csv,console}] [-C CONFIG] [-I] [-V]
```

```
optional arguments:
  -h, --help            show this help message and exit
  -C CONFIG, --config CONFIG
                        Name of config file
  -I, --initialise      Initialise SBOM manager
  -V, --version         show program's version number and exit

Input:
  -a ADD_FILE, --add ADD_FILE
                        SBOM file to be added
  -t {spdx,cyclonedx,csv,dir}, --sbom-type {spdx,cyclonedx,csv,dir}
                        SBOM file type
  -l {all,sbom,module}, --list {all,sbom,module}
                        list SBOMs (default all)
  -m MODULE, --module MODULE
                        Find module in SBOMs
  -d DESCRIPTION, --description DESCRIPTION
                        Description of SBOM file
  -p PROJECT, --project PROJECT
                        Project name
  -s, --scan            Scan SBOMs for vulnerabilities

Output:
  -q, --quiet           Suppress output
  -L {debug,info,warning,error,critical}, --log {debug,info,warning,error,critical}
                        Log level (default: info)
  -o OUTPUT_FILE, --output-file OUTPUT_FILE
                        Output filename (default: output to stdout)
  -f {csv,console}, --format {csv,console}
                        Output format (default: console)
```
						
## Operation

To start using the tool, a repository needs to be created.

`python sbom.py -I`

You can also use this command if the repository needs to be reset, e.g. following an upgrade to the tool.

Once a repository is created, SBOM files can be added. The following types of SBOMs are supported:

  - SPDX (Tag/Value format compatible with version SPDX 2.2).
  - CycloneDX (XML format).
  - CSV where the file is a set of lines containing vendor, product, version entries.
  - DIR which is a file containing a directory listing of filenames. To create a directory file on a Linux based system, the following command can be used `find . -type f > dir_list`

The type of SBOM to be added is specified using the `--type` parameter. A spdx tag value file will be generated, if required, for each SBOM file.

The `--project` parameter is intended to be used to allow for filtering of SBOMs when querying for data. 

If the `--description` parameter is not specified when adding SBOM files, a default value of 'Not Specified' is assumed. This parameter is typically
intended to be used to record build versions of a project.

Invalid entries in an SBOM file will be silently ignored although specifying `--Log debug` may provide some insight into what is being processed.

The `--module` option is used to query the repostory for the existence of a particular module. This may optionally be filtered by project name. The
name of the module is assumed to be wildcard so that a search for a module called 'lib' will find all modules which contain the sequence 'lib'.

The `--list` option is used to report the contents of the repository. 

The `--config` option is used to specify the [configuration file](#configuration-file) to be used. This is required when the  `--scan` is specified.

The `--scan` option is used to scan a SBOM for vulnerabilities. This requires the use of an external vulnerability scanner which
takes a spdx tag value file as input. The vulnerability scanner to be used in specified in the [configuration file](#configuration-file)
specified in the `--config` option.

The `--output-file` and `--format` options can be used to control the formatting and destination of the output generated by the tool. The
default is to report to the console but can be stored in a file (specified using `--output-file` option). The format of the output can be changed using 
the `--format` option which may be useful if the output is to be used as an input by another tool.

## Configuration File

A configuration file is used to specify a number of options for the tool.

```
# SBOM configuration file
[data]
# Use default value if not specified
#location = ""
[scan]
application = cve-bin-tool
# Options are dependent on application. Typically used to define output format or debug levels
options = --sbom spdx --sbom-file
```

Comments are indicated by lines beggining with '#'. All content is ignored

The following options are supported:

- *data* is used to specify the location of the repository to store the SBOM files. A default location is used if this is not specified.

- *application* is used to specify the name of the application to be used with the `--scan` option. A fully qualified path may need to be specified
depending on the system configuration.

- *options* is used to specify any application specific options to be used when scanning a SBOM file for vulnerabilities. The SBOM file name to be scanned
will be automatically appended to the options.

## Licence

Licenced under the MIT Licence

## Limitations

This tool is meant to support software development and security audit functions. However the usefulness of the tool is dependent on the SBOM data
which is provided to the tool. Unfortunately, the tool is unable to determine the validity or completeness of such a SBOM file; users of the tool
are therefore reminded that they should assert the quality of the data before adding any data to the tool. 

## Use Cases

Typical use cases for the tool are:

  - Is my organisation impacted by vulnerability Z with component X?
  - Does my project use version X of component Y?
  - What version(s) of component Y is being used?
  - What vulnerabilities exist within my product? And what needs to be fixed?

### Is my organisation impacted by vulnerability Z with component X?

`python sbom.py -–module <xx>`

### Does my project use version X of component Y?

`python sbom.py –p <project name> -m <product name> | grep <version>`

### What version(s) of component Y is being used?

To look across all projects

`python sbom.py -m <product name>`

This can also be filtered on a project basis.

`python sbom.py –p <project name> -m <product name>`

### What vulnerabilities exist within my product? And what needs to be fixed?

This requires the use of an external vulnerability scanner which takes a spdx tag value file as input. The path
to the vulnerability scanner is specified in a configuration file as well as any tool specific parameters to be
specified (e.g. to filter on severity value).

`python sbom.py –p <project name> --scan`

## Feedback and Contributions

Bugs and feature requests can be made via GitHub Issues. Take care when providing output to make sure you are not
disclosing security issues in other products.

Pull requests are via git.

## Security Issues

Security issues with the tool itself can be reported using GitHub Issues.

If in the course of using this tool you discover a security issue with someone else's code, please disclose responsibly to the appropriate party.
