# vrs-python

[![PyPI version](https://badge.fury.io/py/ga4gh.vrs.svg)](https://pypi.org/project/ga4gh.vrs)
[![Travis](https://travis-ci.org/ga4gh/vrs-python.svg?branch=master)](https://travis-ci.org/ga4gh/vrs-python)

vrs-python provides Python language support for the [GA4GH Variation
Representation Specification
(VRS)](https://github.com/ga4gh/vrs).

This repository contains several related components:

- **ga4gh.core package** Python language support for certain nascent standards
  in GA4GH. Eventually, this package should be moved to a distinct repo.

- **ga4gh.vrs package** Python language support for VRS.

- **ga4gh.vrs.extras package** Python language support for additional
  functionality, including translating from and to other variant formats and a
  REST service to similar functionality. `ga4gh.vrs.extras` requires access to
  supporting data, as described below.

- **Jupyter notebooks** Demonstrations of the functionality of `ga4gh.vrs` and
  `ga4gh.vrs.extras` in the form of easy-to-read notebooks.

# VRS-Python and VRS Version Correspondence

The ga4gh/vrs-python repo embeds the ga4gh/vrs repo as a git submodule, and
therefore each ga4gh.vrs package on PyPi embeds a particular version of VRS. The
correspondences between the packages may be summarized as:

| vrs-python branch | vrs branch |
| --- | --- |
| [main](https://github.com/ga4gh/vrs-python/tree/main) | [main](https://github.com/ga4gh/vrs/tree/main) |
| [0.6](https://github.com/ga4gh/vrs-python/tree/0.6) | [1.1](https://github.com/ga4gh/vrs/tree/1.1) |
| [0.7](https://github.com/ga4gh/vrs-python/tree/0.7) | [1.2](https://github.com/ga4gh/vrs/tree/1.2) |
| [0.8](https://github.com/ga4gh/vrs-python/tree/0.8) | [1.3](https://github.com/ga4gh/vrs/tree/1.3) |
| [0.9](https://github.com/ga4gh/vrs-python/tree/0.9) | [metaschema-update](https://github.com/ga4gh/vrs/tree/metaschema-update) |

⚠ **Developers: See the development section below for recommendations for using submodules
gracefully (and without causing problems for others!).**

# Installation

## Installing with pip

    pip install ga4gh.vrs[extras]

The `[extras]` argument tells pip to install packages to fullfill the dependencies of the
`ga4gh.vrs.extras` package.

## Installing dependencies for ga4gh.vrs.extras

Note : The ga4gh.vrs.extras package used in this project has dependencies on services like *SeqRepo*. To simplify the setup process and ensure consistency across different environments, **Docker** is used to manage these dependencies. Docker allows you to package the required services and their configurations into containers, making it easier to deploy and run them on any system.



#### Step 1: Install Docker
To begin, you need to install Docker on your machine. Follow the official Docker installation guide for your specific operating system:

[Docker Installation Guide](https://docs.docker.com/get-docker/)

### Step 2: Verify Docker Installation
Once Docker is installed, you can verify the installation by running the following command in your terminal:


``` docker version ```

This command displays the Docker client and server versions, indicating that Docker is successfully installed on your machine.

### Step 3: Run a Docker Container
To test Docker, let's run a simple container. In your terminal, execute the following command:

``` docker run hello-world ```

This command pulls the hello-world Docker image (if not already present) and runs it in a container. You should see a message confirming that Docker is working correctly.

[Refer Documentation](https://docs.docker.com/)




The `ga4gh.vrs.extras` modules are not part of the VR spec per se. They are
bundled with ga4gh.vrs for development and installation convenience. These
modules depend directly and indrectly on external data sources of sequences,
transcripts, and genome-transcript alignments. This section recommends one way
to install the biocommons tools that provide these data.

    docker volume create --name=uta_vol
    docker volume create --name=seqrepo_vol
    docker-compose up

This should start three containers:

- [seqrepo](https://github.com/biocommons/seqrepo): downloads seqrepo into a
  docker volume and exits
- [seqrepo-rest-service](https://github.com/biocommons/seqrepo-rest-service): a
  REST service on seqrepo (localhost:5000)
- [uta](https://github.com/biocommons/uta): a database of transcripts and
  alignments (localhost:5432)

Check that the containers are running:

    $ docker ps
    CONTAINER ID        IMAGE                                    //  NAMES
    86e872ab0c69        biocommons/seqrepo-rest-service:latest   //  vrs-python_seqrepo-rest-service_1
    a40576b8cf1f        biocommons/uta:uta_20180821              //  vrs-python_uta_1

Depending on your network and host, the _first_ run is likely to take 5-15
minutes in order to download and install data. Subsequent startups should be
nearly instantaneous.

You can test UTA and seqrepo installations like so:

    snafu$ psql -XAt postgres://anonymous@localhost/uta -c 'select count(*) from transcript'
    249909

### It doesn't work!

Here are some things to try.

- Bring up one service at a time. For example, if you haven't download seqrepo
  yet, you might see this:

      snafu$ docker-compose up seqrepo-rest-service
      Starting vrs-python_seqrepo-rest-service_1 ... done
      Attaching to vrs-python_seqrepo-rest-service_1
      seqrepo-rest-service_1  | 2022-07-26 15:59:59 snafu seqrepo_rest_service.__main__[1] INFO Using seqrepo_dir='/usr/local/share/seqrepo/2021-01-29' from command line
      ⋮
      seqrepo-rest-service_1  | OSError: Unable to open SeqRepo directory /usr/local/share/seqrepo/2021-01-29
      vrs-python_seqrepo-rest-service_1 exited with code 1

# Running the Notebooks

Once installed as described above, type

    $ source venv/3.7/bin/activate
    $ jupyter notebook --notebook-dir notebooks/

The following jupyter extensions are recommended but not required

    $ pip install jupyter_contrib_nbextensions
    $ jupyter contrib nbextension install --user
    $ jupyter nbextension enable toc2/main

## Running the Notebooks on the Terra platform

[Terra](https://terra.bio) is a cloud platform for biomedical research developed by the Broad Institute, Microsoft and Verily. The platform includes preconfigured environments that provide user-friendly access to various applications commonly used in bioinformatics, including Jupyter Notebooks.

We have created a public [`VRS-demo-notebooks`](https://app.terra.bio/#workspaces/terra-outreach/VRS-demo-notebooks) workspace in Terra that contains the demo notebooks along with instructions for running them with minimal setup. To get started, see either the [`VRS-demo-notebooks`](https://app.terra.bio/#workspaces/terra-outreach/VRS-demo-notebooks) workspace or the [`Terra.ipynb`](notebooks/Terra.ipynb) notebook in this repository.


# Development

## Submodules!

vrs-python embeds vrs as a submodule. When checking out vrs-python and switching
branches, it is important to make sure that the submodule tracks vrs-python
correctly. The recommended way to do this is `git config --global submodule.recurse true`. **If you don't set submodule.recurse, developers and
reviewers must be extremely careful to not accidentially upgrade or downgrade
schemas with respect to vrs-python.**

Alternatively, see `misc/githooks/`.

## Installing for development

Fork the repo at https://github.com/ga4gh/vrs-python/ .

    $ git clone --recurse-submodules git@github.com:YOUR_GITHUB_ID/vrs-python.git
    $ cd vrs-python
    $ make devready

## Pre-commit

We use [pre-commit](https://pre-commit.com/) to check code style.

Before first commit, run:

    $ pre-commit install

## Testing

This package implements typical unit tests for ga4gh.core and ga4gh.vrs. This
package also implements the compliance tests from vrs (vrs/validation) in the
tests/validation/ directory.

    $ make test

## Developing VRS (the schema) too

If you want to develop the VRS schema in conjunction with vrs-python, the
recommended approach for most users is to fork and clone the `ga4gh/vrs` repo,
then set the `VRS_SCHEMA_DIR` environment variable to use an alternative schema
location.

# Security Note (from the GA4GH Security Team)

A stand-alone security review has been performed on the specification itself.
This implementation is offered as-is, and without any security guarantees. It
will need an independent security review before it can be considered ready for
use in security-critical applications. If you integrate this code into your
application it is AT YOUR OWN RISK AND RESPONSIBILITY to arrange for a security
audit.
