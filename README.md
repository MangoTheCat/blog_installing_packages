# Installing Packages without Internet
Graham Parsons  



At Mango we're often giving R training in locations where a reliable WiFi connection is not always guaranteed, so if we need trainees to download packages from CRAN it can be a show-stopper.

Here are a couple of code snippets that are useful to download packages from CRAN onto a USB stick when you have a good connection and then install them on site from the USB should you need to.

## In the Office: Download the Dependencies

Knowing the packages we need is one thing, but knowing which packages they depend on is another, and knowing which packages those dependencies depend on is... well, not worth thinking about -- there's a function that comes with R to do it for us called `package_dependencies()`.

Here's a short example script that uses `package_dependencies()` to figure out the dependencies from the packages we want to use.


```r
#' Get package dependencies
#'
#' @param packs A string vector of package names
#'
#' @return A string vector with packs plus the names of any dependencies
getDependencies <- function(packs){
  dependencyNames <- unlist(
    tools::package_dependencies(packages = packs, db = available.packages(), 
                                which = c("Depends", "Imports"),
                                recursive = TRUE))
  packageNames <- union(packs, dependencyNames)
  packageNames
}
# Calculate dependencies
packages <- getDependencies(c("tidyverse", "mangoTraining"))
```
We can then download the right package type for the environment we're going to be training. Often our customers are on Windows so we would download the "win.binary" type. We're also going to save the package *file names* too so that we can install them by filename later.


```r
# Download the packages to the working directory.
# Package names and filenames are returned in a matrix.
setwd("D:/my_usb/packages/")
pkgInfo <- download.packages(pkgs = packages, destdir = getwd(), type = "win.binary")
# Save just the package file names (basename() strips off the full paths leaving just the filename)
write.csv(file = "pkgFilenames.csv", basename(pkgInfo[, 2]), row.names = FALSE)
```

## On Site: Install the Packages

Assuming we've downloaded our packages to a USB stick or similar, on site and without an internet connection we can now install the packages from disk.


```r
# Set working directory to the location of the package files
setwd("D:/my_usb/packages/")

# Read the package filenames and install
pkgFilenames <- read.csv("pkgFilenames.csv", stringsAsFactors = FALSE)[, 1]
install.packages(pkgFilenames, repos = NULL, type = "win.binary")
```
