.. -*- encoding: utf-8 -*-

.. |kJ/mol/nm**2| replace:: kJ mol\ :sup:`-1` nm\ :sup:`-2`
.. |Calpha| replace:: C\ :sub:`α`


==================================
Generate a solvated protein system
==================================


Generate a topology
===================

Using the modified PDB file (chain A of 4AKE with crystal waters removed),
generate a topology file for the CHARMM27 force field together with the
TIP3P water model using the pdb2gmx_ tool::

    cd top
    gmx pdb2gmx -f ../coord/4ake_a.pdb -o protein.pdb -p 4ake.top -i protein_posre.itp -water tip3p -ff charmm27

.. Note:: Total charge -4.000e (in the next step we will add ions to
          neutralize the system; we need a net-neutral system)


Solvate the protein
===================

Adding water
------------

Create a simulation box with editconf_ and add solvent with `solvate`_::

  cd ../solvation
  gmx editconf -f ../top/protein.pdb -o boxed.pdb -c -d 1.2 -bt dodecahedron
  gmx solvate -cp boxed.pdb -cs spc216 -p ../top/4ake.top -o solvated.pdb

.. Note:: In order to reduce the system size and make the simulations run
          faster we are choosing a very tight box (minimum protein-edge
          distance 0.5 nm, ``-d 0.5``); for simulations you want to publish
          this number should be 1.2...1.5 nm so that the electrostatic
          interactions between copies of the protein across periodic
          boundaries are sufficiently screened.

solvate_ updates the number of solvent molecules ("SOL") in the
topology file (check the ``[ system ]`` section in
``top/system.top``) [#topupdate]_.


Adding ions
-----------

Ions can be added with the genion_ program in Gromacs.

First, we need a basic TPR file (an empty file is sufficient, just
ignore the warnings that :program:`grompp` spits out by setting
``-maxwarn 10``), then run :program:`genion` (which has convenient
options to neutralize the system and set the concentration (check
the help!); :program:`genion` also updates the topology's ``[ system
]`` section if the top file is provided [#topupdate]_; it reduces the
"SOL" molecules by the number of removed molecules and adds the
ions, e.g. "NA" and "CL"). ::

  touch ions.mdp
  gmx grompp -f ions.mdp -p ../top/4ake.top -c solvated.pdb -o ions.tpr
  printf "SOL" | gmx genion -s ions.tpr -p ../top/4ake.top -pname NA -nname CL -neutral -conc 0.15 -o ionized.pdb

The final output is :file:`solvation/ionized.pdb`. Check visually in VMD
(but note that the dodecahedral box is not represented properly.)


.. _`pdb_downloader.sh`:
   http://becksteinlab.physics.asu.edu/pages/courses/2013/SimBioNano/03/pdb_downloader.sh
.. _Practical 2:
   http://becksteinlab.physics.asu.edu/pages/courses/2013/SimBioNano/02/

.. _`AdKTutorial.tar.bz2`:
    http://becksteinlab.physics.asu.edu/pages/courses/2013/SimBioNano/13/AdKTutorial.tar.bz2
.. _4AKE: http://www.rcsb.org/pdb/explore.do?structureId=4ake
.. _pdb2gmx: http://manual.gromacs.org/current/online/pdb2gmx.html
.. _editconf: http://manual.gromacs.org/current/online/editconf.html
.. _solvate: http://manual.gromacs.org/current/online/solvate.html
.. _genion: http://manual.gromacs.org/current/online/genion.html
.. _trjconv: http://manual.gromacs.org/current/online/trjconv.html
.. _trjcat: http://manual.gromacs.org/current/online/trjcat.html
.. _eneconv: http://manual.gromacs.org/current/online/eneconv.html
.. _grompp: http://manual.gromacs.org/current/online/grompp.html
.. _mdrun: http://manual.gromacs.org/current/online/mdrun.html
.. _`mdp options`: http://manual.gromacs.org/current/online/mdp_opt.html
.. _`Run control options in the MDP file`: http://manual.gromacs.org/current/online/mdp_opt.html#run
.. _`make_ndx`: http://manual.gromacs.org/current/online/make_ndx.html
.. _`g_tune_pme`: http://manual.gromacs.org/current/online/g_tune_pme.html
.. _gmxcheck: http://manual.gromacs.org/current/online/gmxcheck.html

.. _Gromacs manual: http://manual.gromacs.org/
.. _Gromacs documentation: http://www.gromacs.org/Documentation
.. _`Gromacs 4.5.6 PDF`: http://www.gromacs.org/@api/deki/files/190/=manual-4.5.6.pdf
.. _manual section: http://www.gromacs.org/Documentation/Manual

.. _`g_rms`: http://manual.gromacs.org/current/online/g_rms.html
.. _`g_rmsf`: http://manual.gromacs.org/current/online/g_rmsf.html
.. _`g_gyrate`: http://manual.gromacs.org/current/online/g_gyrate.html
.. _`g_dist`: http://manual.gromacs.org/current/online/g_dist.html
.. _`g_mindist`: http://manual.gromacs.org/current/online/g_mindist.html
.. _`do_dssp`: http://manual.gromacs.org/current/online/do_dssp.html

.. _DSSP: http://swift.cmbi.ru.nl/gv/dssp/
.. _`ATOM record of a PDB file`: http://www.wwpdb.org/documentation/format33/sect9.html#ATOM


.. rubric:: Footnotes

.. [#crystalwaters] Often you would actually want to retain
   crystallographic water molecules as they might have biological
   relevance. In our example this is likely not the case and by
   removing all of them we simplify the preparation step somewhat. If
   you keep them, :program:`pdb2gmx` in the next step will
   actually create entries in the topology for them.

.. [#topupdate] The automatic modification of the top file by
   :program:`solvate` and :program:`genion` can become a problem if you
   try to run these commands multiple times and you get error messages
   later (typically from :program:`grompp`) that the number of
   molecules in structure file and the topology file do not agree. In
   this case you might have to manually delete or adjust the
   corresponding lines in :file"`system.top` file.