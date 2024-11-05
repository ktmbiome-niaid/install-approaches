## Install Approaches

The purpose of this repository is primarily to test out different package installation approaches, especially when some of them are buggy. This will be an improvement to making bajillions of minor commits and pushes to the main training repository. This also means that components of the repository (ie, install.R or apt.txt) should be considered entirely ephemeral.

As much as possible, I'll also update this readme with notes about what I might have learned from the process. This is a public repository, so maybe it will entertain someone besides me one day...

### My Badge

Placing a badge here for easy access...

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/ktmbiome-niaid/install-approaches/HEAD?urlpath=rstudio)

## October 28, 2024

**Problem 1**: The version of Rstudio that comes with Binder by default is too old to allow graphics to be displayed. So deploying a Binder using default settings means things like plots won't be displayed *or* you'll need to back-track your packages.

**Potential Solution**: I learned that you can create Dockerfile setups so that you can pull from rocker/binder the specific R version you want and it will also bring in the right level of Rstudio. The Dockerfile looks something like below and was obtained from [here](https://github.com/yuvipanda/rstudio-binder-template).

```
FROM rocker/binder:4.3.3

## Declares build arguments
ARG NB_USER
ARG NB_UID

COPY --chown=${NB_USER} . ${HOME}

ENV DEBIAN_FRONTEND=noninteractive
USER root
RUN echo "Checking for 'apt.txt'..." \
        ; if test -f "apt.txt" ; then \
        apt-get update --fix-missing > /dev/null\
        && xargs -a apt.txt apt-get install --yes \
        && apt-get clean > /dev/null \
        && rm -rf /var/lib/apt/lists/* \
        ; fi
USER ${NB_USER}

## Run an install.R script, if it exists.
RUN if [ -f install.R ]; then R --quiet -f install.R; fi
```

While I was doing this, I tried a couple of things, including but not limited to:

- Installing conda so that I could use an `environment.yml` to manage my packages and dependencies | Problem was that I didn't have permissions to write to /opt and then it didn't really like me installing conda anyway; this also meant I was playing around with permissions and things as well...

**Problem 2**: This became an issue that required solving after problem 1, but I'm not 100% sure if it existed before. Under the new paradigm, I couldn't install Maaslin2 in the environment. Trying to install it like any other package meant that it wasn't installed but the Rstudio environment was still presented, so you would do something like `library(Maaslin2)` and no package existed to pull up. 

**Potential Solution 1**: It was a specific issue with compilation of RcppEigen within glmmTMB. It *seems* I am able to build by running from *apt*, which is kind of crazy. So creating an apt.txt file and putting r-cran-glmmtmb appears to maybe work. And then installing maaslin2 from the install.R file is at least what I'm trying right now. I'm still having compilation issues, but it's better than the red wall of text I was getting before...

Still not building the container with Maaslin correctly installed...

**Potential Solution 2**: Apt required too many dependencies (I think that's why it was failing -- almost 800MB of additional package dependencies), so I've removed it and am instead installing glmmTMB from the install.R again, possibly with different options based on https://rdrr.io/cran/glmmTMB/man/reinstalling.html.

**Correct Answer**: I went back to R 4.2.0, which then installed an older version of Bioconductor, which then installed older versions of Maaslin2 and its dependencies, which worked with this particular system.
