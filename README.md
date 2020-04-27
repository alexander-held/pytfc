# pytfc

## Table of contents

- [Introduction](#introduction)
- [Template fits](#template-fits)
- [Configuration file thoughts](#configuration-file-thoughts)
- [Scope](#scope)

## Introduction

`pytfc` is (or rather will be) a `py`thonic `t`emplate `f`it `c`aptain, which steers (profile likelihood) template fits with applications in high energy physics in mind.
It acts as an interface to many powerful tools to make it easier for an analyzer to run their statistical inference pipeline.
For the moment, this repository is a collection of things rather than a fully featured product.
An incomplete list of interesting tools to interface:

- [ServiceX](https://github.com/ssl-hep/ServiceX) for data delivery,
- [coffea](https://github.com/CoffeaTeam/coffea) for histogram processing,
- [uproot](https://github.com/scikit-hep/uproot) for reading [ROOT](https://root.cern.ch/) files
- for building likelihood functions (captured in so-called workspaces in [RooFit](https://root.cern.ch/roofit)) and inference:
    - [RooFit](https://root.cern.ch/roofit) to model probability distributions
    - [RooStats](http://cds.cern.ch/record/1289965) for statistical tools
    - [HistFactory](https://cds.cern.ch/record/1456844/) to implement a subset of binned template fits
    - [pyhf](https://github.com/scikit-hep/pyhf) for a pythonic take on [HistFactory](https://cds.cern.ch/record/1456844/),
    - [zfit](https://github.com/zfit/zfit) for a pythonic take on [RooFit](https://root.cern.ch/roofit)
    - [MadMiner](https://github.com/diana-hep/madminer) for likelihood-free inference techniques

The project is a work in progress, and the name is a placeholder.
Configuration fo `pytfc` should happen in a declarative manner, and be easily serializable via JSON/YAML into a configuration file.
Some of the discussion below needs generalization for [MadMiner](https://github.com/diana-hep/madminer) style applications, see also the [Scope](#scope) section.

Interesting related projects:

- [pyhfinput](https://github.com/lukasheinrich/pyhfinput)
- [ServiceX for TRExFitter](https://github.com/kyungeonchoi/ServiceXforTRExFitter)
- [Template fit workflows](https://github.com/alexander-held/template-fit-workflows)
- [TRExFitter config translation](https://github.com/alexander-held/TRExFitter-config-translation)

## Template fits

The operations needed in a template fit workflow can be summarized as follows:

1. [Template histogram production](#1-template-histogram-production),
2. [Histogram adjustments](#2-histogram-adjustments),
3. [Workspace creation from histograms](#3-workspace-creation-from-histograms),
4. [Inference from workspace](#4-inference-from-workspace),
5. [Visualization](#5-visualization).

While the first four points need to happen in this order (as each step uses as input the output of the previous step), the visualization is relevant at all stages to not only show final results, but also intermediate steps.

### 1. Template histogram production

The production of a template histogram requires the following information:

- where to find the data (and how to read it),
- what kind of selection requirements (filtering) and weights to apply to the data,
- the variable to bin in, and what bins to use (for binned fits)
- a unique name (key) for this histogram to be able to refer to it later on.

In practice, histogram information can be given by specifying lists of:

- samples (physics processes),
- regions of phase space (or channels, independent regions obtained via different selection requirements),
- systematic uncertainties for the samples, which might vary across samples and phase space regions.

For LHC-style template profile likelihood fits, typically a few thousand histograms are needed.
An analysis that considers 10 different physics processes (simulated as 10 independent Monte Carlo samples, uses 5 different phase space regions, and an average of 50 systematic uncertainties for all the samples (implemented by specifying variations from the nominal configuration in two directions) needs `10x5x100=5000` systematics.

### 2. Histogram adjustments

Histogram post-processing can include re-binning, smoothing, or symmetrization of systematic uncertainties.
These operations should be handled by tools outside of `pytfc`.
Such tools might either need some additional steering via an additional configuration, or the basic configuration file has to support arbitrary settings to be passed to these tools (depending on what each tool can interpret).

### 3. Workspace creation from histograms

Taking the example of [pyhf](https://github.com/scikit-hep/pyhf), the workspace creation consists of plugging histograms into the right places in a JSON file.
This can be relatively straightforward if the configuration file is very explicit about these assignments.
In practice, it is desirable to support convenience options in the configuration file.
An example is the ability to de-correlate the effect of a systematic uncertainty across different phase space regions via a simple flag.
This means that instead of one nuisance parameter, many nuisance parameters need to be created automatically.
The treatment can become complicated when many such convenience functions interact with each other.

A possible approach is to define a lowest level configuration file format that supports no convenience functions at all and everything specified in a very explicit manner.
Convenience functions could be supported in small frameworks that can read configuration files containing flags for convenience functions, and those small frameworks could convert the configuration file into the low level format.

The basic task of building the workspace should have well-defined inputs (low-level configuration file) and outputs (such as [HistFactory](https://cds.cern.ch/record/1456844/) workspaces).
Support of convenience functions can be factored out, with a well-defined output (low-level configuration file) and input given by an enhanced configuration file format.

### 4. Inference from workspace

Inference happens via fits of the workspace, to obtain best-fit results and uncertainties, limits on parameters, significances of observations and so on.
External tools are called to perform inference, configured as specified by the configuration file.

### 5. Visualization

Some information of relevant kinds of visualization is provided in [as-user-facing/fit-visualization.md](https://github.com/iris-hep/as-user-facing/blob/master/fit-visualization.md) and the links therein.

## Configuration file thoughts

### Grouping of options

The configuration file is how analyzers specify their fit model.
Experience shows that it can get complex quickly.
It is desirable to group configuration settings in ways that can make the file easier to read.
For example, the color with which to draw a sample in figures does not matter for the fit model.
It should be possible to easily hide such options for easier inspection of the configuration file, and this could be achieved by grouping them together as "cosmetics".

### Validation

As much as possible, automatic checks of the configuration file structure and content should happen before running any computationally expensive steps.
For example, if input data is declared to be at various different locations, a quick check could verify that indeed data can be found at the paths declared.
This can quickly flag typos before any histogram production is run.

### Interactions with other existing frameworks

While ambitious, it would be great to be able to translate configurations of other existing frameworks into a `pytfc` configuration, to be able to easily run detailed comparisons.
Some relevant work for [TRExFitter](https://gitlab.cern.ch/TRExStats/TRExFitter) exists [here](https://github.com/alexander-held/TRExFitter-config-translation).

## Scope

Traditional binned template fits in [HistFactory](https://cds.cern.ch/record/1456844/) style are substantially easier to support than the open world of binned and unbinned models.
Likelihood-free inference approaches in  the style of [MadMiner](https://github.com/diana-hep/madminer) have a more well-defined scope than the open world of [RooFit](https://root.cern.ch/roofit), and might be easier to integrate.
