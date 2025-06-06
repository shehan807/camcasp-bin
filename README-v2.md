### Install Camcasp

* Camcasp is a “post SCF” code, meaning it requires another software package to do SCF and feed in orbitals to Camcasp. Use Psi4 for this, so make sure you have a Psi4 version installed via anaconda.  In this document, I will refer to my Psi4 install in anaconda environment “psi4_v1.8”

* Download the Camcasp package/binary from here:

https://github.com/ajmisquitta/camcasp-bin

* The camcasp binary can be downloaded to any path, and you will set environmental variables in order to locate the camcasp binary, and also for camcasp to locate the Psi4 binary when it calls Psi4 to run SCF.  You need to set the following environmental variables in any run script that calls camcasp (these are my versions of the paths…)

```
export SCRATCH=/storage/home/hcoda1/3/jmcdaniel43/scratch
export CAMCASP=/storage/home/hcoda1/3/jmcdaniel43/p-jmcdaniel43-0/rich_project_chem-mcdaniel/Programs/camcasp-7.2
export PATH=$CAMCASP/bin:$PATH
export PSIPATH=/storage/home/hcoda1/3/jmcdaniel43/p-jmcdaniel43-0/rich_project_chem-mcdaniel/Programs/camcasp-7.2/basis/psi4
export PSIPATH=/storage/home/hcoda1/3/jmcdaniel43/p-jmcdaniel43-0/rich_project_chem-mcdaniel/Programs/camcasp-7.2/basis/psi4/for-psi4-lib:$PSIPATH
export PSIPATH=$CAMCASP/basis/psi4:$PSIPATH
export PSI4_HOME=/storage/home/hcoda1/3/jmcdaniel43/.conda/envs/psi4_v1.8/
```

*** NOTE: The final backslash in the PSI4_HOME variable is important to add!!

### Camcasp input file

Create a camcasp input file called a "cluster file" for your calculation, usually this has the extension "*.clt". I will refer to the example "methylacetamide.clt".  The file contains the following:

```
Title davtz properties

!

Global

  Units Angstrom

  Overwrite Yes

End



Molecule methylacetamide

 !

I.P. 0.3489  au

charge 0

C 6 0.97373 -0.6876 0.13713  Type C

N 7 0.64371 0.47895 -0.65813 Type N

C 6 1.59412 1.40187 -0.97422 Type C

O 8  2.75683 1.2806 -0.59993 Type O

C 6 1.10728 2.56719 -1.80927 Type C

H 1 0.06546 -1.29825 0.27404 Type H

H 1 1.35321 -0.39389 1.13028 Type H

H 1 1.73941 -1.30336 -0.36405 Type H

H 1 -0.31462 0.61562 -0.98898 Type H

H 1 0.31876 3.12969 -1.28159 Type H

H 1 1.95386 3.24685 -2.00411 Type H

H 1 0.70499 2.22022 -2.77592 Type H

End



Run

  Properties

  Molecule  methylacetamide

  Basis avtz

  File-prefix methylacetamide

  Functional PBE0

  Kernel ALDAX+CHF

  SCFcode psi4

  Orient files

  Process file

  Interface files

  Sites file

End

Finish

```

* Everything here is pretty self-explanatory.  Lines with “!” are comment lines.  As you will note, the Ionization potential of the molecule has to be input (here it is 0.3489 a.u.).   This is because Camcasp computes a GRAC shift for the response calculation, exactly how Psi4 computes a GRAC shift for DFT-SAPT.  You thus compute/input the ionization potential in exactly the same way that you would for a DFT-SAPT calculation in Psi4.

* You can find more examples of camcasp input files to use as templates here:
https://github.com/ajmisquitta/camcasp-bin/tree/master/examples

* What we need for force field fitting (polarizabilities, dispersion coefficients), are the so-called point-to-point response (or p2p response) calculations.  For polarizabilities, we need these at static frequency (w=0), and for dispersion coefficients we need these at imaginary frequencies (iw).  The examples that pertain to point-to-point response are found in

https://github.com/ajmisquitta/camcasp-bin/tree/master/examples/properties 

and you can read the “README” discussing these examples.

The original paper discussing point-to-point response calculations for fitting polarizabilities/dispersion coefficients is␍
Williams and Stone, J. Chem. Phys. 2003, 119, 4620-4628

You may also want to refer to Alston Misquitta’s wiki page here:␍

https://wiki.ph.qmul.ac.uk/ccmmp/AJMPublic/camcasp/pols-cn 


### Running Camcasp

* The camcasp software consists of a lot of python scripts that control workflow between various binary executables that constitute the different modules of the code.  For whatever reason, some of the python scripts seem to be “broken” (at least for me), so that running camcasp requires several steps/workarounds to get it to work (probably this could be fixed if someone wanted to understand the python run scripts in detail).

* Make sure you load the anaconda environment with psi4, and set the environmental variables as discussed above.  With either an interactive job or slurm submit script, run the command (note this will run Psi4, so make sure it runs on the compute nodes!):

```
runcamcasp.py methylacetamide --clt methylacetamide.clt 
```

* this will run Psi4/camcasp on 1 core, which will be slow.  It would be nice to use more cores.  If you run `runcamcasp.py –help’, it indicates that you can set the number of cores with an argument such as ‘—cores 8’.   However, I tried this and it doesn’t seem to be working.

* The Camcasp calculation will apparently crash after a while.  You will see output such as:

```
Starting CamCASP with 1 threads...
9
CamCASP finished with error code 9 at 13:23:36
Job methylacetamide failed at 13:23:36
``` 

* This means that camcasp has failed, but Psi4 has run correctly which was the first necessary part of the calculation.  What you need to do now is obtain the required output of Psi4 from the scratch directory, fix the camcasp file, and then rerun it (and hopefully it works).

* Go to the scratch directory where the camcasp calculation ran.  In my case, this is in 

```
/storage/home/hcoda1/3/jmcdaniel43/scratch/methylacetamide_01
```

* If you look in the directory, you will see a Psi4 output file, in my case called “methylacetamide_A.out”.  Look at this file with vi to make sure the Psi4 calculation executed correctly.

* Now assuming this Psi4 output file looks ok, there should be a directory called `camcasp’ with the required input files for camcasp (as generated by Psi4).  The two key files are the file which contains the molecular orbital information “*.movecs” extension, in my case this is called “methylacetamide-A-asc.movecs”, and the file which contains the basis set information, in my case this is “methylacetamide-A.basis”.  Copy both of these files to the top scratch directory, /storage/home/hcoda1/3/jmcdaniel43/scratch/methylacetamide_01

* The ‘runcamcasp.py’ script generated a new “*.cks” input file that is used to directly run the camcasp binary.  The reason the camcasp calculation crashed is that the paths to the basis function files (in Psi4) are not listed correctly in this file (I don’t know why runcamcasp.py  messes this up?).  So you need to go in and manually fix the paths.  If you open this file, in my case “methylacetamide.cks”

If you look in this file you will see a section with the Auxiliary basis functions.  It should have entries for all the atoms which look like

```
     C          6.0        1.84008289      -1.29937559       0.25913812  TYPE C
        Limit G
        #include-camcasp basis/auxiliary/aug-cc-pVTZ/C
```

* The reason the calculation crashed, is that this path doesn’t work to find the auxiliarly basis.  Change the path to point directly to the basis file in the camcasp download, in my case this is

```
        #include /storage/home/hcoda1/3/jmcdaniel43/p-jmcdaniel43-0/rich_project_chem-mcdaniel/Programs/camcasp-7.2/basis/auxiliary/aug-cc-pVTZ/C
```

* You need to do this for all the atoms in the file.  Once you have done this, the calculation “should work”.  In the same scratch directly (and making sure you are on a compute node), directly run the camcasp binary with a command like

```
/storage/home/hcoda1/3/jmcdaniel43/p-jmcdaniel43-0/rich_project_chem-mcdaniel/Programs/camcasp-7.2/bin/camcasp < methylacetamide.cks
```

The camcasp calculation should now run.


### Camcasp Output

* When the camcasp calculation finishes, you should see a lot of different output files in the scratch directory.  The file that we need is the “point-to-point” response, which should have a “*.p2p” extension.  Make sure this file exists, and make sure that it is fairly large (should be several hundred MB).

* Most of the output describing the camcasp response calculation was printed to standard out, so it’s not a bad idea to capture the standard output when you run the camcasp binary, by piping to an output file , 

```
e.g. /path/camcasp < methylacetamide.cks > camcasp.output
```

* the last thing we need to do is separate the *.p2p file into different files that correspond to different frequency information.  I used to know how to do this, but the executable has changed.  See the “localize.py” file in https://github.com/ajmisquitta/camcasp-bin/tree/master/bin, and you basically need to run python code equivalent to lines 271-288, which reads

```
    if not ok:
        p2pfile = findfile("OUT",".p2p")
        if not p2pfile:
            die("Can't find a point-to-point (.p2p) file")
        print("Splitting the point-to-point polarizability file. Please wait ... ")
        if verbosity > 0: print(f"Using p2p file {p2pfile}")
        sys.stdout.flush()
        with open("split_p2p","w") as SPLIT:
            SPLIT.write(f"""read P2P pols for {name}
    Frequencies Static + 10
    P2P-Pols {p2pfile} SPLIT
  End
  """)
        if verbosity > 0:
            subprocess.call(["process < split_p2p"], shell=True)
        else:
            subprocess.call(["process < split_p2p > /dev/null"], shell=True)
        os.remove("split_p2p")
```
