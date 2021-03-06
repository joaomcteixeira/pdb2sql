.. ipython:: python
    :suppress:

    # change working dir to docs/
    import os
    os.chdir('..')

=====================
10 minutes to pdb2sql
=====================

This is a short introduction to pdb2sql.


Download PDB files
------------------

A handy tool `fetch` is provided to download PDB files from `PDB <https://www.rcsb.org>`_ website.

.. ipython:: python

    from pdb2sql import fetch
    fetch('3CRO', './pdb/')
    ls ./pdb

For clear illustration, some `dummy PDB files <https://github.com/DeepRank/pdb2sql/tree/master/docs/pdb>`_
are used in the following examples.

Get and set data
----------------

First, we import as follows:

.. ipython:: python

    from pdb2sql import pdb2sql

Create a SQL database instance:

.. ipython:: python

    db = pdb2sql("./pdb/dummy.pdb")


The ``db`` is a SQL instance that contains one table named *ATOM*.

In this table, each row represents one atom, and columns are atom properties:

.. ipython:: python

    db.print()

Get data
^^^^^^^^

Get chainID, residue number, residue name and atom name of all atoms:

.. ipython:: python

    p = db.get('chainID, resSeq, resName, name')
    p

Get x,y,z coordinates of all atoms:

.. ipython:: python

    p = db.get('x,y,z')
    p

Get x,y,z coordinates of chain A atoms:

.. ipython:: python

    p = db.get('chainID, x,y,z', chainID=['A'])
    p

Get x,y,z coordinates of atoms on residue 1 and 4 of Chain A

.. ipython:: python

    p = db.get('chainID,resSeq,x,y,z', chainID=['A'], resSeq=['1', '4'])
    p

Get data of all atoms except residue MET and GLN atoms

.. ipython:: python

    p = db.get('chainID, resSeq, resName, name', no_resName = ['MET', 'GLN'])
    p

Get data of all atoms except residue MET and GLN atoms or CA (carbon alpha) atoms

.. ipython:: python

    p = db.get('chainID, resSeq, resName, name', no_resName = ['MET', 'GLN'], no_name = ['CA'])
    p


Get all data, a simple way is ``db.get('*')``.

A shortcut to get x,y,z coordinates:

.. ipython:: python

    p = db.get_xyz()
    p

Get chain IDs:

.. ipython:: python

    p = db.get_chains()
    p

Get residue list:

.. ipython:: python

    p = db.get_residues()
    p


Filter the data base
^^^^^^^^^^^^^^^^^^^^^^^^^^

pdb2sql allows to create a new database by filtering the one we jut created

.. ipython:: python

    db_chainA = db(chainID='A')
    db_chainA.print()

In that example `dp_chainA` is a sql database that only includes the atoms from chain A.
All the selection keywords (chainID, resSeq, resName, name) and their negations 
(no_chainID, no_resSeq, no_resName, no_name) can be used and combined to obtain the new database.

Set data
^^^^^^^^

Rename chain B to C:

.. ipython:: python

    num_B_atoms = len(db.get('chainID', chainID=['B']))
    chainC = ['C'] * num_B_atoms
    db.get_chains()
    db.update('chainID', chainC, chainID = ['B'])
    db.get_chains()


Update x,y,z coordinates for structure translatation of [10,10,10]

.. ipython:: python

    xyz_old = db.get_xyz()
    xyz = np.array(xyz_old) + 10.0
    db.update('x,y,z', xyz)
    xyz_new = db.get_xyz()
    print("old:\n", xyz_old)
    print("new:\n", xyz_new)

Update a column using index, e.g. change the x coordinates of the first
10 atoms to 2:

.. ipython:: python

    x = np.ones(10) + 1
    db.update_column('x', values=x, index=list(range(10)))
    db.print('serial, name, x')

Add a new column *type* with value *high*:

.. ipython:: python

    db.add_column('type', value = 'high', coltype = 'str')
    db.print('serial, name, type')


PDB I/O
-------

Read PDB file or data to a list:

.. ipython:: python

    pdb = pdb2sql.read_pdb('./pdb/dummy.pdb')
    pdb

Convert SQL data to PDB-formated data:

.. ipython:: python

    pdb = db.sql2pdb()
    pdb

Write PDB file from SQL database:

.. ipython:: python

    db.exportpdb('./pdb/test.pdb')

    # show the test.pdb file
    ls ./pdb



Interface calculation
---------------------

Create an :class:`~pdb2sql.interface.interface` SQL database instance:

.. ipython:: python

    from pdb2sql import interface

    # use pdb2sql instance as input
    from pdb2sql import pdb2sql
    pdb_db = pdb2sql('./pdb/3CRO.pdb')
    db = interface(pdb_db)

    # or use pdb file as input
    db = interface('./pdb/3CRO.pdb')

Interface atoms
^^^^^^^^^^^^^^^

.. ipython:: python

    itf_atom = db.get_contact_atoms(cutoff = 3)
    itf_atom_pair = db.get_contact_atoms(cutoff = 3, return_contact_pairs=True)
    print("interface atom:\n", itf_atom)
    print("interface atom pairs:\n", itf_atom_pair)


Interface residues
^^^^^^^^^^^^^^^^^^

.. ipython:: python

    itf_residue = db.get_contact_residues(cutoff = 3)
    itf_residue_pair = db.get_contact_residues(cutoff = 3, return_contact_pairs=True)
    itf_residue
    itf_residue_pair


Structure superposition  
--------------------------

pdb2sql allows to superpose two structure on top of each other either using the full structure or with selection keywords.
For example to superpose the chain A of two PDB one can use :

.. ipython:: python 

    from pdb2sql import superpose
    ref = pdb2sql('./pdb/1AK4_5w.pdb')
    decoy = pdb2sql('./pdb/1AK4_10w.pdb')
    superposed_decoy = superpose(decoy, ref, chainID='A', export=True)

This will export a new PDB file containining the structure of the decoy superposed onto the reference.

Structure alignement
---------------------------

pdb2sql allows to align structure along a specific axis

.. ipython:: python

    from pdb2sql import align
    db = pdb2sql('./pdb/1AK4_10w.pdb')
    aligned_db = align(db, axis='z', export=True)

The alignement can  also consider only a subpart of the complex using the selection keywords:

.. ipython:: python

    aligned_db = align(db, axis='z', chainID='A')

There the chain A will be aligned along the z-axis

This will create a new PDB file containing the structure aligned along the z-axis. It is 
also possible aligning an interface in a given plane

.. ipython:: python

    from pdb2sql import align_interface
    db = pdb2sql('./pdb/3CRO.pdb')
    aligned_db = align_interface(db, plane='xy', export=True)

By default the interface formed by chain A and B will be considered. In case multiple chains are present
in the structure it is possible to specify wich interface to consider:

.. ipython:: python

    aligned_db = align_interface(db, plane='xy', chain1='L', chain2='R')


There the interface between chain L and R will be considered. Note that any other selection
keyword can be used to specify which interface to account for.

Structure similarity calculation
--------------------------------

Create a :class:`~pdb2sql.StructureSimilarity.StructureSimilarity` instance:

.. ipython:: python

    from pdb2sql.StructureSimilarity import StructureSimilarity
    sim = StructureSimilarity('./pdb/decoy.pdb', './pdb/ref.pdb')

interface RMSD
^^^^^^^^^^^^^^

.. ipython:: python
    :okwarning:

    irmsd_fast = sim.compute_irmsd_fast()
    irmsd_pdb2sql = sim.compute_irmsd_pdb2sql()
    irmsd_fast
    irmsd_pdb2sql


ligand RMSD
^^^^^^^^^^^

.. ipython:: python
    :okwarning:

    lrmsd_fast = sim.compute_lrmsd_fast()
    lrmsd_pdb2sql = sim.compute_lrmsd_pdb2sql()
    lrmsd_fast
    lrmsd_pdb2sql

FNAT
^^^^

Calculate the fraction of native contacts:

.. ipython:: python
    :okwarning:

    fnat_fast = sim.compute_fnat_fast()
    fnat_pdb2sql = sim.compute_fnat_pdb2sql()
    fnat_fast
    fnat_pdb2sql


DockQ score
^^^^^^^^^^^

.. ipython:: python

    dockQ = sim.compute_DockQScore(fnat_fast, lrmsd_fast, irmsd_fast)
    dockQ


Structure transformation
------------------------

Create SQL instance:

.. ipython:: python

    from pdb2sql import transform
    db = pdb2sql('./pdb/dummy_transform.pdb')

The atom coordinates are:

.. ipython:: python

    db.get_xyz()

Rotations
^^^^^^^^^
Rotate structures 180 degrees along the x-axis:

.. ipython:: python

    angle = np.pi
    axis = (1., 0., 0.)
    transform.rot_axis(db, axis, angle)
    db.get_xyz()

Get random rotation axis and angle:

.. ipython:: python

    axis, angle = transform.get_rot_axis_angle()
    axis
    angle

Translations
^^^^^^^^^^^^

Translate structure 5Å along y-axis:

.. ipython:: python

        trans_vec = np.array([0,5,0])
        transform.translation(db, trans_vec)
        db.get_xyz()