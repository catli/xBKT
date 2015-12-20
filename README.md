Created by Zachary A. Pardos (zp@berkeley.edu) and Matthew J. Johnson (mattjj@csail.mit.edu)
Computational Approaches to Human Learning Research (CAHL) Lab @ UC Berkeley
# Running #

You can work in the repository root directory or add it to your path with
`addpath` (no need to use `genpath`, since everything is organized with
namespaces).

See the file `+test/hand_specified_model.m` for a fairly complete example,
which you can run with `test.hand_specified_model`.

If you get an error about missing the function `E_step`, the compiling step
(see "Installation and setup") didn't work.

Here's a simplified version:

```matlab
%% generate synthetic model and data
% the model will have 4 question subparts
num_subparts = 4;
truemodel = generate.random_model(num_subparts);

% generate 3 observation sequences with varying lengths
data = generate.synthetic_data(truemodel,[200,150,500]);

%% fit models, starting with random initializations
best_likelihood = -inf;
for i=1:25
    [fitmodel, log_likelihoods] = fit.EM_fit(generate.random_model(num_subparts),data);
    if (log_likelihoods(end) > best_likelihood)
        best_likelihood = log_likelihoods(end);
        best_model = fitmodel;
    end
end

%% compare the fit model to the true model
disp('these two should look similar');
truemodel.A
best_model.A
```

# Installation and setup #

## Cloning the repository ##

```
git clone git@github.com/CAHLR/xBKT.git
```

## Installing Eigen ##

Get Eigen from http://eigen.tuxfamily.org/index.php?title=Main_Page and unzip
it somewhere (anywhere will work, but it affects the mex command below). On a
\*nix machine, these commands should put Eigen in /usr/local/include:


    cd /usr/local/include
    wget --no-check-certificate http://bitbucket.org/eigen/eigen/get/3.1.3.tar.gz
    tar -xzvf 3.1.3.tar.gz
    ln -s eigen-eigen-2249f9c22fe8/Eigen ./Eigen
    rm 3.1.3.tar.gz

Similarly, if working in OS X, you can download the latest stable version of Eigen 
from the site above. This program has run successfully with `Eigen 3.2.5`.
First move the file to /usr/local/include, then unzip and create simplified link to Eigen. 
These commands can be used below:


    mv <path to file>/3.1.3.tar.gz /usr/local/include/3.1.3.tar.gz
    tar -xvf 3.1.3.tar.gz
    ln -s <name of unzipped file>/Eigen ./Eigen
    rm 3.1.3.tar.gz


## Compiling ##

Run `make` in the root directory of the xBKT project folder. If this step runs successfully, you should see a MEX file generated for each of the .cpp files. 

## Potential Errors When Running Makefile on OS X ##

Before running `make`, check `Makefile` in xBKT. Be sure that the `MATLABPATH` matches your matlab version and `EIGENPATH` matches your Eigen filepath. For example, if you're working with Matlab 2015 in OS X, you may need to update `Makefile` with the new name of your `Applications` from

```
    ifeq ($(UNAME),Darwin)
        MATLABPATH=/Applications/MATLAB_R2013a.app
    endif
```

to something like


```
    ifeq ($(UNAME),Darwin)
        MATLABPATH=/Applications/MATLAB_R2015b.app
    endif
```    


You may see the following error while running `make`
```
    make: g++-4.9: No such file or directory
```

Try `gcc --version` in your terminal. If a version exists, you already have gcc installed. This error may be due to an incorrect version of gcc being called. In order to change the gcc version in `Makefile`, update the `CXX` variable. For example, you may need to change `CXX=g++-4.9` to `CXX=g++-5`, depending on the version you set up. 

If a version does not exist, you  may need to download gcc49. This can be downloaded with [brew](http://brew.sh/). 

These steps would allow you to set up gcc49. Run the following commands
```
    brew install --enable-cxx gcc49
    brew install mpfr
    brew install gmp
    brew install libmpc
```

## Preparing Data for xBKT ##
xBKT models student mastery of a skills as they work through a series of learning resources and checks for understanding. At each checkpoint, students may be given a resource (learning practice activity) and/or question(s) to check for understanding. The model finds the probability of learning, forgetting, slipping and guessing that maximizes the likelihood of observed student response to questions. 

To run the xBKT model, define the following variables:
* `num_subparts`: The number of unique questions used to check understanding. Each sub-part has a unique emissions matrix.
* `num_resources`: The number of resources shown to students
* `num_fit_initialization`: The number of iterations in the EM step

Next, create a `Data` object, containing the input for the model. `Data` contains the following attributes: 
* `data`: a matrix containing sequential checkpoints for all students, with their responses. Each row represents a different subpart, and each column a checkpoint for a student. There are three potential values: {0 = no response or no question asked, 1 = wrong response, 2 = correct response}. If at a checkpoint, a resource was given but no question asked, the column would have all `0` values. For example to set up data for two students with two and three checkpoints respectively and 5 subparts, the matrix would be as follows:

    | 0 | 0 | 0 | 0 | 2 |

    | 0  1  0  0  0 |
    | 0  0  0  0  0 |
    | 0  0  0  0  0 |
    | 0  0  2  0  0 |   

In this example, each student start with a resource but no responses. In subsequent responses, both student eventually answered questions correctly. 
* `starts`: defines each student's starting position on the `data` matrix. 
* `lengths`: defines the number of check point for each student. 



