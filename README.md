# Fig2code: create runnable, reproducible figures

Fig2code is a free, open-source tool to create figures that contain all the information needed to run them.  

## Motivation

A typical figure shared within a company, or in a scientific manuscript is not backed up with the code, data and environment that created it. This can lead to loss in productivity, wrong conclusions and knowledge being trapped on personal computers.  

Fig2code's goal is to make it easy for data scientists of all skill levels to create runnable, reproducible figures with minimal effort. 

Fig2code is based on Docker, but no knowledge of Docker is needed to use it. It abstracts all the Docker details to focus on the operations meaningful for data analysis workflows. 

Fig2code supports R and R markdown. Support for python and jupyter is planed for the near future. 

## Comparison with Jupyter / Markdown / Sweave

Jupyter notebooks and markdown are a way to keep the code and the output together as produced on a single machine. Fig2code wraps this and other types of code into isolated runnable units that are guaranteed to run the same on all machines. Compared to notebooks, fig2code also tracks the data, dependencies and any other required external files. 

## When to use

Continue using R interactively to create analysis and experiment. Once you are ready to produce the final figures use fig2code to create them, so they are backed up by code, data and environment.  Share your final figures with collaborators and the public. 

Fig2code is not meant as a replacement for the R and python packaging systems. If the primary purpose of your code is to be distributed as a module to be re-used by others, please use the existing packaging systems. 

## Usage

### Basic usage

To create a runnable figure, write in the terminal:

```
fig2code run myScript.R 
```

This is going to do the following:

- examine  `myScript.R` file for dependencies and select the most appropriate base Docker container.
- run `myScript.R` inside Docker, isolate the code files that are necessary to run the figure and capture the outputs. 
- create an HTML version of the script output, e.g. If the script outputs `myFigure.png`, it will create `myFigure.html`. Meta information needed to re-run the figure, such as the Dockerfile, code and (small) data is recorded in hidden HTML tags. This meta information is designed to enlarge the file only minimally. Large files are recorded by their SHA256 value.

When opened in a browser the fig2code output HTML looks like a figure, but contains the full information need to re-run the code on any computer.

You can also access this function from the Rstudio fig2code add-in. 

For most users, this is the only command that you will need to know. 

### Advanced usage

Runnable figures come with a lot of useful information you can use to understand figures and manage pipelines. 

#### 1. Listing the content of a runnable figure

List the meta data store in the runnable figure:

```
fig2code list myFigure.html
```

This will return the list of meta data stored, e.g. code files, etc. 

See an individual files stored in the figure with:

```
fig2code cat myFigure.html myScript.R
```

This will show the content of the `myScript.R` file in the console. 

#### 2. Unpack a runnable figure

Unpack all the stored files into a separate directory:

```
fig2code unpack myFigure.html
```

This will create a directory `myFigure_src/` that will contain all the necessary files to re-run the figure. 

#### 3. See if any of the input files have changed
 
See if any of the files needed for the figure have changed:

```
fig2code diff myFigure.html
```

If the figure has been created by `myScript.R`, this command will check if the versions stored in `myFigure.html` and the current directory are the same. If not, it will show a diff file. 

#### 4. Test if the same figure is produced

Test if an identical figure will be produced by the current version of the script. 

```
fig2code test myScript.R
```

This will run `myScript.R` and compare the output, e.g. `myFigure.html` with the version of the file that was created before. If there are differences, this will be reported. 
 
#### 5. Examine the figure generation
 
```
fig2code examine myFigure.html
```

This will re-run the figure and start the R debugger at the end of the R script. You can examine the R workspace and run R commands. 

Start the R debugger at an arbitrary point in the R code:

```
fig2code examine -file myScript.R -line 1 myFigure.html
```
 
#### 6. Re-run a figure

```
fig2code run myFigure.html
```

This will use the meta information stored in `myFigure.html`, provision any necessary Docker containers and re-run the figure inside Docker. 

This is useful when the script generating the figure takes parameters:

```
fig2code run myFigure.html <argument 1> <argument 2>
```

or when the figure is based on Shiny, in which case a shiny server will be started and the user redirected to the browser.

If re-running the figure requires a large file, it will be located automatically using the fig2code SHA256 cache, that can be generated with:

```
fig2code add largeFile.rda
```

This adds `largeFile.rda` to the SHA256 cache, so it can be automatically found when needed to re-run a figure.  

#### 7. Create the figure remotely

You can create the figure remotely (e.g. in the cloud). To do so you will need to have fig2code and Docker installed on the remote machine. 

```
fig2core remote-run <server address> myScript.R . 
```

When using this command fig2code will use SSH to connect to the remote server and run the `myScript.R` command. The last argument is the path of the context directory, all files in this directory will be uploaded unless they already exist at the destination server.

## Handling of data files

Scripts might output data files such as CSV or RData files. These files are not useful when embedded into HTML, so for these, the provenance information is stored in the central cache repository. Each output is identified with its path and SHA256, and all the files that would be normally embedded are attached to this hash. These attached files can themselves be hashes to avoid duplication in the repository. 

Using fig2code along the whole pipeline then enables full provenance, from the initial raw files to the final figure. 

## Configuration

Configure fig2code globally for the user by changing the `~/.fig2code.yml` file, which can be overriden by a `fig2code.yml` file in the current directory. 

Configuration options:

- `big_file:5mb` - specifies the total max size of data files to embed. If data files exceed this limit, they won't be embedded. 

## Licence

Apache-2