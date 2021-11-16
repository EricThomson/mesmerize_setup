# mesmerize installation instructions
Recipe I use when installing Mesmerize on Windows. Basically follows instructions at the Mesmerize web site:
http://docs.mesmerizelab.org/en/master/user_guides/installation.html

I install this fairly frequently, this is a recipe that works well even for machines without a lot of RAM.



# A. Initial stuff
1. Install miniconda     
Just follow the instructions here:
https://docs.conda.io/en/latest/miniconda.html
Select the option to add anaconda to the PATH environment during installation if you want to make life simpler. If you already have it installed (or anaconda), then skip this step.

2. Initialize powershell support
Open your anaconda prompt and initialize Windows Powershell support:

    ```
    conda init powershell
    ```
Close your anaconda prompt, as *the rest of your steps will be in powershell*. You will notice when you open Windows Powershell now, you will have conda access (i.e., you will be in a conda base virtual environment with `(base)` before the directory path).

3. Change some powershell policies    
Run Windows Powershell as admin and enter the following:
```
    Set-ExecutionPolicy RemoteSigned    
    Set-ExecutionPolicy Bypass -scope Process -Force
```
Then close powershell.

4. Increase pagefile size       
(Again this is for Windows). This step effectively gives you more RAM/for when you run CNMF-E. [Follow the instructions here](
http://www.tomshardware.com/faq/id-2864547/manage-virtual-memory-pagefile-windows.html). The recommended minimum pagefile size is 1.5x your actual RAM, and I recommend (at least) 64GB for the maximum value, if your computer will allow it.  Note you will have to restart your computer after this step.

What if you already have more than 64GB of RAM? Then I am not 100% sure this step is needed. For such machines, then I'd suggest skipping this step, and coming back to it only if you hit problems with RAM/out of memory errors, and try setting it to 64GB-128GB if you can (though frankly this is a rule of thumb that depends on how much RAM you have: I am not an expert on this we might want to talk to the caiman people about this at some point). 

# B. Install and configure Mesmerize
1. Create virtual environment, activate, and install stuff (do the following all in powershell):    

        conda update conda
        conda install setuptools
        conda create -n mesmerize python=3.6

        conda activate mesmerize
        conda install mamba -n mesmerize -c conda-forge
        mamba install -c anaconda tensorflow-gpu=1.15 # if you don't have an nvidia gpu just install tensorflow
        mamba install caiman -c conda-forge  #can take a long time ~20 minutes sometimes
        mamba install Cython
        mamba install -c conda-forge tslearn=0.4.1 bottleneck=1.2 graphviz
        pip install mesmerize  # tadaaaa!

2. Create your mesmerize directory  
Set up a directory where you will want to do your mesmerize stuff, preferably a place with *lots* of space (ideally more than a TB). It will be helpful to run mesmerize from there. Let's call this `mesmerize_path`

3. Complete setup and configuration   
cd into `mesmerize_path` and run mesmerize with the command `mesmerize` from your virtual environment:
```
    cd mesmerize_path
    conda activate mesmerize
    mesmerize
```
The first time you run it, mesmerize will download a bunch of things and go through some final setup steps that can take quite a few minutes.

4. Create a working directory    
Once done, that setup step will have created the following `.mesmerize` directory in your home directory:

          `C:/users/username/.mesmerize`

Create a new directory, `working_dir` in that subdirectory so you now have:

         `C:\Users\username\.mesmerize\working_dir`

5. Finish setup configuration      
You will have the mesmerize gui open now. Go into `configuration-> system configuration`, and paste the working directory path you just created (`C:\Users\username\.mesmerize\working_dir`) into the `working_dir` text field.

Add the following to the console:
```
          set MKL_NUM_THREADS=1
          set OPENBLAS_NUM_THREADS=1
```
Put those before `conda activate mesmerize` (and *uncomment* that line).    

Set the number of threads to use (like half or so of the number of logical cores you have).

6. Fix the `border_px` problem

You need to make it so that bord_px doesn't screw up your `corr_pnr` plots for cnmfe. Open the following:    

      C:\Users\username\miniconda3\envs\mesmerize\Lib\site-packages\mesmerize\viewer\modules\cnmfe.py
      
(Note if already had conda installed on your system in Step A above, then the environment/packages may be in a different location: just find the above file and open it).  

Replace this (around line 49)
```
        try:
            bord_px = next(d for ix, d in enumerate(history_trace) if 'caiman_motion_correction' in d)['caiman_motion_correction']['bord_px']
        except StopIteration:
            bord_px = 0
```
With this:
```
        bord_px = 0
```
Now this will just force border pix to be 0, and you won't have any funky artifacts.

Close mesmerize. It is now configured and ready to run!

# D. Give it a test run!
In powershell, cd into `mesmerize_path` and start mesmerize and get going: create a new project, and start doing things (presumably you have some test data you want to play with). Just follow the video tutorials:    
https://www.youtube.com/playlist?list=PLgofWiw2s4REPxH8bx8wZo_6ca435OKqg

## Things to do:
- Load movie
- Test out motion correction batch (make sure you have good params!)
- Test out cnmfe batch (again make sure you have good params!)
- Voila!


### Example params
You will need to tweak for your movie.

motion:

    gsigfilt 5
    max_shifts 14
    max_devition_rigid 14
    overlaps 90 strides 45


cnmfe:

    ssub 2
    tsub 1
    min_corr .5
    min_pnr 2.5
    gSig 4, gSiz 17
    patch.rf 80
    patch.stride 40
    ssubB2
    nProcesses [huh what is this?]

# E. Additional stuff
1. To upgrade mesmerize:
    pip install mesmerize --update

2. From mesmerize to Jupyter
For saving and loading between mesmerize and jupyter, see:
https://github.com/kushalkolar/MESmerize/blob/master/notebooks/demo_batch_ingestion.ipynb


# Acknowledgments
- Kushal Koler for the amazing Mesmerize package.
- Caiman for providing support and the use of the sample data to test the installation on. 
- Developed with the support from NIH Bioinformatics and the Neurobehavioral Core at NIEHS. Thanks to the many people there who were patient while I nailed down this install on their computers.  
