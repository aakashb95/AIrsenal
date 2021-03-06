# AIrsenal
[![Build Status](https://travis-ci.org/alan-turing-institute/AIrsenal.svg?branch=master)](https://travis-ci.org/alan-turing-institute/AIrsenal)

*AIrsenal* is a package for using Machine learning to pick a Fantasy Premier League team.

For some background information and details see https://www.turing.ac.uk/research/research-programmes/research-engineering/programme-articles/airsenal.

We welcome contributions and comments - if you'd like to join the AIrsenal community please refer to our [contribution guidelines](https://github.com/alan-turing-institute/AIrsenal/blob/master/CONTRIBUTING.md)

## Pre-requisites

The Stan model used to predict match results is in the package https://github.com/anguswilliams91/bpl, and to run this you will need a working (recent) C++ compiler. To test you have gcc installed in your system run the following command in a terminal:
```
gcc --version
```

If this successfully returns version information you can continue with the AIrsenal installation process. If not you will need to install `gcc`. Common ways to do this include:
* **Mac OSX:** `brew install gcc`
* **Linux (Ubuntu):** `apt-get install build-essential`
* **Windows:** We recommend using conda and following the windows-specific instructions below. Alternativley, have a look at [MinGW](http://www.mingw.org/wiki/Getting_Started) to get a working compiler.

Alternatively, please refer to the Cython installation pre-requirements for options to get a working compiler on your system here: http://docs.cython.org/en/latest/src/quickstart/install.html.


## Install

We recommend running AIrsenal in a conda environment. For instructions on how to install conda go to this link: https://docs.anaconda.com/anaconda/install/

With conda installed, run these commands in a terminal to create a new conda environment and download and install AIrsenal:

**Linux and Mac OS X:**
```
conda create -n airsenalenv python=3.7 pystan
conda activate airsenalenv
git clone https://github.com/alan-turing-institute/AIrsenal.git
cd AIrsenal
pip install .
```

**Windows:**

_Windows is not fully supported. You should be able to install the module but there are still compatibility issues (see issue [#165](https://github.com/alan-turing-institute/AIrsenal/issues/165)). You may have more success trying to run AIrsenal on the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/about) instead._

If you already have `gcc` working on your system you can follow the Linux & Mac OS X instructions above. Otherwise try the steps below based on the [pystan documentation](https://pystan.readthedocs.io/en/latest/windows.html):
```
conda create -n airsenalenv python=3.7
conda activate airsenalenv
conda install libpython m2w64-toolchain -c msys2
conda install numpy cython pystan -c conda-forge
git clone https://github.com/alan-turing-institute/AIrsenal.git
cd AIrsenal
pip install .
```

AIrsenal is under regular development both to fix problems that come up and add new functionality. If you find a bug or have ideas for improvements or new features

## Getting started

Once you've installed the module, you will need to set some environment variables (or alternatively you can put the values into files in the ```airsenal/data/``` directory, e.g. ```airsenal/data/FPL_TEAM_ID```:

**Required:**
1. `FPL_TEAM_ID`: the team ID for your FPL side.

**Optional:**

2. `FD_API_KEY`: an API key for [football data](https://www.football-data.org/) (this is only needed for filling past seasons results if not already present as a csv file in the ```data/``` directory.)

3. `FPL_LEAGUE_ID`: a league ID for FPL (this is only required for plotting FPL league standings).

4. `FPL_LOGIN`: your FPL login, usually email (this is only required to get FPL league standings).

5. `FPL_PASSWORD`: your FPL password (this is only required to get FPL league standings).

Once this is done, run the following command:

```shell
airsenal_setup_initial_db
```

You should get a file ```/tmp/data.db```.  This will fill the database with all that is needed up to the present day.

You can run sanity checks on the data using the following command:

```
airsenal_check_data
```

## Updating, running predictions and optimization.

To stay up to date in the future, you will need to fill three tables: ```match```, ```player_score```, and ```transaction```
with more recent data, using the command
```shell
airsenal_update_db
```

The next step is to use the team- and player-level Stan models to predict the expected points for all players for the next fixtures.  This is done using the command
```shell
airsenal_run_prediction --weeks_ahead 3
```
(we normally look 3 weeks ahead, as this is an achievable horizon to run the optimization over, but also because things like form and injuries can change a lot in 3 weeks!)

Finally, we need to run the optimizer to pick the best transfer strategy over the next weeks (and hence the best team for the next week).
```shell
airsenal_run_optimization --weeks_ahead 3
```
This will take a while, but should eventually provide a printout of the optimal transfer strategy, in addition to the teamsheet for the next match (including who to make captain, and the order of the substitutes).

Note that `airsenal_run_optimization` should only be used for transfer suggestions after the season has started. If it's before the season has started and you want to generate a full squad for gameweek one you should instead use:
```shell
airsenal_make_team --num_gw 3
```

Run any of the commands above with the `--help` flag to see additional options (to allow wildcards or other chips to be used, for example).

## Issues and Development

AIrsenal is regularly developed to fix bugs and add new features. If you have any problems during installation or usage please let us know by [creating an issue](https://github.com/alan-turing-institute/AIrsenal/issues/new) (or have a look through [existing issues](https://github.com/alan-turing-institute/AIrsenal/issues) to see if it's something we're already working on).

You may also like to try the development version of AIrsenal, which has the latest fixes and features. To do this checkout the `develop` branch of the repo before the `pip install .` step of the installation instructions above by running the following command:
```
git checkout develop
```

Also, if you wish to make changes to AIrsenal yourself it may be helpful to install AIrsenal in editable mode by adding the `-e` flag to `pip`:
```
pip install -e .
```

## Contributing

We welcome all types of contribution to AIrsenal, for example questions, documentation, bug fixes, new features and more. Please see our [contributing guidelines](CONTRIBUTING.md). If you're contributing for the first time but not sure what to do a good place to start may be to look at our [current issues](https://github.com/alan-turing-institute/AIrsenal/issues), particularly any with the ["Good first issue" tag](https://github.com/alan-turing-institute/AIrsenal/issues?q=is%3Aopen+is%3Aissue+label%3A%22good+first+issue%22). Also feel free to just say hello!

