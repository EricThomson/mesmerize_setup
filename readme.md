# mesmerize installation instructions
Getting mesmerize working on Windows:
http://docs.mesmerizelab.org/en/master/user_guides/installation.html

I install this fairly frequently, this is a recipe that works well even for machines without a lot of RAM. Use at your own risk!

Mesmerize written by Kushal Kolar:
https://github.com/kushalkolar/MESmerize

# A. Initial stuff

1. Install miniconda (select the option to add anaconda to the PATH environment during installation)
2. Install Atom (not really necessary but useful).
3. Open your anaconda prompt:

    conda init powershell

4. The rest of your steps will be in powershell, go open powershell don't do anything else in conda prompt.

# B. Create virtual environment
Note if computer doesn't have gpu replace tensorflow-gpu with tensorflow in the following.
1. First change some settings in powershell, run as admin:    

    >Set-ExecutionPolicy RemoteSigned    
    Set-ExecutionPolicy Bypass -scope Process -Force

    Then close powershell, and open as normal (non-admin).

2. Increase pagefile    
This effectively gives you more RAM/memmap file size. It requires admin privileges so since you are doing step 1 already, might as well do this now too:    
http://www.tomshardware.com/faq/id-2864547/manage-virtual-memory-pagefile-windows.html

  Reasoning: the Windows memmap interface is sensitive to the maximum setting and leaving it at the default can cause errors when processing larger datasets.
  https://caiman.readthedocs.io/en/master/Installation.html#installing-caiman
  
  You will have to restart your computer after this step.

3. Create virtual environment, activate, and install stuff:    

        conda update conda
        conda install setuptools
        conda create -n mesmerize python=3.6
        conda activate mesmerize

        conda install mamba -n mesmerize -c conda-forge
        mamba install -c anaconda tensorflow-gpu=1.15 #takes several minutes
        mamba install tensorflow-estimator=1.15    #probably not needed check first
        mamba install caiman -c conda-forge  #takes a long time ~20 minutes or more
        mamba install Cython
        mamba install -c conda-forge tslearn=0.4.1 bottleneck=1.2 graphviz
        pip install pywin32  # probably not needed
        pip install mesmerize  # tadaaaa!

# C. Configure and setup finalize
1. Set up directory    
Set up a directory where you will want to do your mesmerize stuff, preferably a place with *lots* of space (ideally more than a TB). It will be helpful to run mesmerize from there. Let's call this `mesmerize_path`

2. Complete setup and configuration    
a. Run mesmerize    
 cd into `mesmerize_path` and run mesmerize with the command `mesmerize`. The first time you run it, it will download a bunch of things and go through some final setup steps that will take a few minutes.

  b. Create a working directory    
  Once done, that setup step will have created the following `.mesmerize` directory:

      `C:/users/username/.mesmerize`

  Create a new directory, `working_dir` in that subdirectory so you now have:

  	 `C:\Users\username\.mesmerize\working_dir`

  c. System configuration    
  i. You will have mesmerize gui open now. Go into `configuration-> system configuration`, and paste the working directory path you just created (`C:\Users\username\.mesmerize\working_dir`) into the `working_dir` text field.

  ii. Add the following to the console:
  
      set MKL_NUM_THREADS=1
      set OPENBLAS_NUM_THREADS=1
      
  Put those before `conda activate mesmerize` (and *uncomment* that line).
  iii. Set the number of threads to use (like half or so of the number of logical cores you have).

  See relevant github issue if this is confusing at all:
  https://github.com/kushalkolar/MESmerize/issues/53

3. fix the problem with border_px
You need to make it so that bord_px doesn't screw up your plots. Open the following:
      C:\Users\lab_user\miniconda3\envs\mesmerize\Lib\site-packages\mesmerize\viewer\modules\cnmfe.py

      Replace this (around line 49)

              try:
                  bord_px = next(d for ix, d in enumerate(history_trace) if 'caiman_motion_correction' in d)['caiman_motion_correction']['bord_px']
              except StopIteration:
                  bord_px = 0

    With this:

              bord_px = 0

  Now this will just force border pix to be 0. :)

  Close mesmerize.

# D. Give it a test run!
Create two new folders in `mesmerize_path`: `projects` and `data`. Drop a data file you want to analyze in the data file. In powershell, cd into `mesmerize_path` and start mesmerize and get going: create a new project, and start doing things. Just follow the video tutorials:
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