---
title: Setup
---


## Software Setup

:::::::::::::::: solution

### Windows

You can watch the [YouTube video tutorial](https://www.youtube.com/watch?v=q0PjTAylwoU) for complete instructions.

- [ ] Install R by downloading and running [this .exe file](http://cran.r-project.org/bin/windows/base/release.htm") from [CRAN](http://cran.r-project.org/index.html).  
- [ ] Please, also [install Rtools](https://cran.r-project.org/bin/windows/Rtools/rtools42/rtools.html)
  - Note that if you have separate user and admin accounts, you should run the installers as administrator (right-click on `.exe` file and select "Run as administrator" instead of double-clicking). Otherwise problems may occur later,  for example when installing R packages.
:::::::::::::::::::::::::

:::::::::::::::: solution

### MacOS

Follow the [video tutorial](https://www.youtube.com/watch?v=5-ly3kyxwEg) for detailed instructions.

- [ ] Install R by downloading and running [this .pkg file](http://cran.r-project.org/bin/macosx/R-latest.pkg) from [CRAN](http://cran.r-project.org/index.html)

:::::::::::::::::::::::::


:::::::::::::::: solution

### Linux

- [ ] You can download the binary files for your distribution
        from [CRAN](http://cran.r-project.org/index.html

**Or**

- [ ] you can use your package manager (e.g. for Debian/Ubuntu

```
sudo apt-get install r-base
```

- [ ] for Fedora run
```
sudo dnf install R
```
:::::::::::::::::::::::::



**There are some extra things to install for all operating systems.**
<br><br>

::::::::::::::::::::::::: solution

### RStudio

Please install the [RStudio IDE](http://www.rstudio.com/ide/download/desktop). 
It is the user interface towards R, and is required for this workshop.

::::::::::::::::::::::::: 

::::::::::::::::::::::::: solution

### R packages 
Lastly, you will need to install two packages to join the workshop, namely the {tidyverse} and {palmerpenguins} packages.
You can do this by opening RStudio, and in the panel labelled "console" (usually in the bottom left corner), type the following:

```r
install.packages(c("tidyverse", "palmerpenguins"))
```

::::::::::::::::::::::::: 
