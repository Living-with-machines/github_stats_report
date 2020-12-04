# Living with Machines GitHub Stats report

![Create Report](https://github.com/davanstrien/lwm_github_stats_report/workflows/Create%20Report/badge.svg)

> “Facts are stubborn things, but statistics are pliable.” ― Mark Twain (probably)

## tl;dr

This repository automatically updates GitHub statistics data for the Living with Machines [GitHub Organization](https://github.com/Living-with-machines) and generates a report based on this data.

Report link \ # TODO add link to report output

## What

GitHub provides statistics for repositories which include views and clones traffic. However, by default, this information is only shown for two weeks. This repository uses [GitHub Actions](https://github.com/features/actions) and [gh_orgstats](https://github.com/Living-with-machines/gh_orgstats) to grab data every week and update a CSV file for public repositories under the Living with Machines GitHub Organization. You can find this documented in more tedious detail below.

Alongside updating the CSV files, this repository also uses the magic of [Jupyter notebooks](https://jupyter.org/) and [nbconvert](https://github.com/jupyter/nbconvert) to update a report based on these GitHub stats automatically.

### The Data

You can find the view and clone data for public Living with Machines repositories in the following folders:

- Data for repository views: [view_data](view_data/)
- Data for repository clones: [clone_data](clone_data/)

### The Report

The report is made up of three parts:

- [`report.ipynb`](report.ipynb) is a Jupyter notebook that uses a combination of [gh_orgstats](https://github.com/Living-with-machines/gh_orgstats), [Pandas](pandas.pydata.org/) and [Altair](altair-viz.github.io/) to print out some DataFrames and some more or less ugly charts. This notebook defines the report content and calls two methods in `gh_orgstats` that update the CSV files for clone and view stats. 

- [`report.nbconvert.ipynb`](report.nbconvert.ipynb) is the executed output version of the above notebook. This execution happens using [nbconvert](https://github.com/jupyter/nbconvert) and GitHub actions. Again, see below for tedious detail. 

- [`report.html`](docs/report.html) is an HTML version of the report which is generated using the output of `report.nbconvert.ipynb`. This HTML output only includes the notebook output, hiding the ugly code from the end-user. The rendered version of this report can be found at \# TOOD add link to rendered HTML

## Why

Living with Machines is a research project funded by the [AHRC](https://ahrc.ukri.org/funding/research/) as part of the [UK Research and Innovation (UKRI](https://www.ukri.org/) Strategic Priority Fund. Research funders usually want to know that the money they have spent on a project is having an impact. One (very imperfect) way of assessing this is via 'usage stats'. Since capturing these stats can be a time consuming process, this repository aims to automate some of the steps.

As a research project aiming to practice openness as far as possible, we decided to make these stats publicly available.

## How the report is generated

For the interested reader, this section outlines how the report is generated in more detail.

### Jupyter notebooks and nbconvert

Jupyter notebooks are an excellent tool for generating reports. You can easily combine prose, statistics, and charts in one place. However, sharing reports in notebooks isn't suitable for all circumstances. The code cells used to define chart, or select data might be distracting for your non-coding colleagues. The code may also upset your colleagues who do code for different reasons. You may therefore want to hide the input cells in the notebook and only share the output cells.

This is where [nbconvert](https://github.com/jupyter/nbconvert) becomes useful. [nbconvert](https://github.com/jupyter/nbconvert) allows you to convert notebooks between different formats, and optionally strip parts of the notebook out. nbconvert can convert notebooks into HTML, markdown, pdf etc. We can also use [nbconvert](https://github.com/jupyter/nbconvert) to 'execute' a notebook. i.e. to run a notebook.

In this example converting to HTML allows us to retain some of the interactivity of the notebooks whilst removing the need for a Python environment for the notebook to run in.

One remaining issue, in this case, is that we want our report to update weekly so we can pull new stats from GitHub (which are only stored for two weeks by default). We could set ourselves a calendar reminder to run the report every week but this isn't a great use of our time. This is where ~~an ensemble of three neural network models~~ GitHub actions become useful.

### GitHub Actions

GitHub actions are a feature of GitHub that allows you to automate various workflows. For example, running tests on pushed changes, creating a welcoming comment the first time someone creates an issue or makes a pull request in an open-source repository. In this case we use an `on schedule` trigger to execute every week.

When this action is triggered the following things happen:

- an environment is setup (in this case Ubuntu).
- we checkout this GitHub repository.
- Jupyter notebook, nbconvert and other packages we need to execute the `report.ipynb` notebook are installed.
- we use `nbconvert` to execute our report notebook and convert it to HTML.
- executing the notebook also has the effect of updating our CSV files of traffic and clones data.
- we commit these updated files back to the repository.

If you already know how to use GitHub actions you can see the workflow for this in [report.yml](.github/workflows/report.yml). If you haven't come across GitHub actions before you can find an [annotated version](action_overview.md) of the workflow which explains the process. 


> *Credit: This project, funded by the UK Research and Innovation (UKRI) Strategic Priority Fund, is a multidisciplinary collaboration delivered by the Arts and Humanities Research Council (AHRC), with The Alan Turing Institute, the British Library and the Universities of Cambridge, East Anglia, Exeter, and Queen Mary University of London.*
