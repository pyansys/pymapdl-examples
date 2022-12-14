
.. DO NOT EDIT.
.. THIS FILE WAS AUTOMATICALLY GENERATED BY SPHINX-GALLERY.
.. TO MAKE CHANGES, EDIT THE SOURCE PYTHON FILE:
.. "vm-001-statically_indeterminate_reaction_force_analysis.py"
.. LINE NUMBERS ARE GIVEN BELOW.

.. only:: html

    .. note::
        :class: sphx-glr-download-link-note

        Click :ref:`here <sphx_glr_download_vm-001-statically_indeterminate_reaction_force_analysis.py>`
        to download the full example code

.. rst-class:: sphx-glr-example-title

.. _sphx_glr_vm-001-statically_indeterminate_reaction_force_analysis.py:


.. _ref_statically_indeterminate_example:

Statically Indeterminate Reaction Force Analysis
------------------------------------------------
Problem Description:
 - A prismatical bar with built-in ends is loaded axially at two
   intermediate cross sections.  Determine the reactions :math:`R_1`
   and :math:`R_2`.

Reference:
 - S. Timoshenko, Strength of Materials, Part I, Elementary Theory and
   Problems, 3rd Edition, D. Van Nostrand Co., Inc., New York, NY, 1955,
   pg. 26, problem 10.

Analysis Type(s):
 - Static Analysis ``ANTYPE=0``

Element Type(s):
 - 3-D Spar (or Truss) Elements (LINK180)

.. image:: _static/vm1_setup.png
   :width: 400
   :alt: VM1 Problem Sketch

Material Properties
 - :math:`E = 30 \cdot 10^6 psi`

Geometric Properties:
 - :math:`a = b = 0.3`
 - :math:`l = 10 in`

Loading:
 - :math:`F_1 = 2*F_2 = 1000 lb`

Analytical Equations:
 - :math:`P = R_1 + R_2` where :math:`P` is load.
 - :math:`\frac{R_2}{R_1} = \frac{a}{b}`
   Where :math:`a` and :math:`b` are the ratios of distances between
   the load and the wall.

.. GENERATED FROM PYTHON SOURCE LINES 43-55

.. code-block:: default

    # sphinx_gallery_thumbnail_path = '_static/vm1_setup.png'

    from ansys.mapdl.core import launch_mapdl

    # start mapdl and clear it
    mapdl = launch_mapdl()
    mapdl.clear()  # optional as MAPDL just started

    # enter verification example mode and the pre-processing routine
    mapdl.verify()
    mapdl.prep7()





.. rst-class:: sphx-glr-script-out

 .. code-block:: none


    *****MAPDL VERIFICATION RUN ONLY*****
         DO NOT USE RESULTS FOR PRODUCTION

              ***** MAPDL ANALYSIS DEFINITION (PREP7) *****



.. GENERATED FROM PYTHON SOURCE LINES 56-60

Define Material
~~~~~~~~~~~~~~~
Set up the material and its type (a single material, with a linking-type
section and a Young's modulus of 30e6).

.. GENERATED FROM PYTHON SOURCE LINES 60-67

.. code-block:: default


    mapdl.antype("STATIC")
    mapdl.et(1, "LINK180")
    mapdl.sectype(1, "LINK")
    mapdl.secdata(1)
    mapdl.mp("EX", 1, 30e6)





.. rst-class:: sphx-glr-script-out

 .. code-block:: none


    MATERIAL          1     EX   =  0.3000000E+08



.. GENERATED FROM PYTHON SOURCE LINES 68-72

Define Geometry
~~~~~~~~~~~~~~~
Set up the nodes and elements.  This creates a mesh just like in the
problem setup.

.. GENERATED FROM PYTHON SOURCE LINES 72-81

.. code-block:: default


    mapdl.n(1, 0, 0)
    mapdl.n(2, 0, 4)
    mapdl.n(3, 0, 7)
    mapdl.n(4, 0, 10)
    mapdl.e(1, 2)
    mapdl.egen(3, 1, 1)






.. rst-class:: sphx-glr-script-out

 .. code-block:: none


    GENERATE       3 TOTAL SETS OF ELEMENTS WITH NODE INCREMENT OF         1
       SET IS SELECTED ELEMENTS IN RANGE         1 TO         1 IN STEPS OF       1

     MAXIMUM ELEMENT NUMBER=         3



.. GENERATED FROM PYTHON SOURCE LINES 82-90

Define Boundary Conditions
~~~~~~~~~~~~~~~~~~~~~~~~~~
Full constrain nodes 1 and 4, by incrementing from node 1 to node 4
in steps of 3. Apply y-direction forces to nodes 2 and 3, with
values of -500 lb and -1000 lb respectively. Then exit prep7.

Effectiely, this sets:
- :math:`F_1 = 2*F_2 = 1000 lb`

.. GENERATED FROM PYTHON SOURCE LINES 90-97

.. code-block:: default


    mapdl.d(1, "ALL", "", "", 4, 3)
    mapdl.f(2, "FY", -500)
    mapdl.f(3, "FY", -1000)
    mapdl.finish()






.. rst-class:: sphx-glr-script-out

 .. code-block:: none


    ***** ROUTINE COMPLETED *****  CP =         0.000



.. GENERATED FROM PYTHON SOURCE LINES 98-101

Solve
~~~~~
Enter solution mode and solve the system.

.. GENERATED FROM PYTHON SOURCE LINES 101-106

.. code-block:: default


    mapdl.run("/SOLU")
    out = mapdl.solve()
    mapdl.finish()





.. rst-class:: sphx-glr-script-out

 .. code-block:: none


    FINISH SOLUTION PROCESSING


     ***** ROUTINE COMPLETED *****  CP =         0.000



.. GENERATED FROM PYTHON SOURCE LINES 107-112

Post-processing
~~~~~~~~~~~~~~~
Enter post-processing. Select the nodes at ``y=10`` and ``y=0``, and
sum the forces there. Then store the y-components in two variables:
``reaction_1`` and ``reaction_2``.

.. GENERATED FROM PYTHON SOURCE LINES 112-122

.. code-block:: default


    mapdl.post1()
    mapdl.nsel("S", "LOC", "Y", 10)
    mapdl.fsum()
    reaction_1 = mapdl.get("REAC_1", "FSUM", "", "ITEM", "FY")
    mapdl.nsel("S", "LOC", "Y", 0)
    mapdl.fsum()
    reaction_2 = mapdl.get("REAC_2", "FSUM", "", "ITEM", "FY")









.. GENERATED FROM PYTHON SOURCE LINES 123-134

Check Results
~~~~~~~~~~~~~
Now that we have the reaction forces we can compare them to the
expected values of 900 lbs and 600 lbs for reactions 1 and 2 respectively.

Analytical results obtained from:
- :math:`P = R_1 + R_2` where :math:`P` is load of 1500 lbs
- :math:`\frac{R_2}{R_1} = \frac{a}{b}`

Hint: Solve for each reaction force independently.


.. GENERATED FROM PYTHON SOURCE LINES 134-144

.. code-block:: default

    results = f"""
        ---------------------  RESULTS COMPARISON  ---------------------
        |   TARGET   |   Mechanical APDL   |   RATIO
        /INPUT FILE=    LINE=       0
        R1, lb          900.0       {abs(reaction_1)}   {abs(reaction_1) / 900}
        R2, lb          600.0       {abs(reaction_2)}   {abs(reaction_2) / 600}
        ----------------------------------------------------------------
        """
    print(results)





.. rst-class:: sphx-glr-script-out

 .. code-block:: none


        ---------------------  RESULTS COMPARISON  ---------------------
        |   TARGET   |   Mechanical APDL   |   RATIO
        /INPUT FILE=    LINE=       0
        R1, lb          900.0       900.0   1.0
        R2, lb          600.0       600.0   1.0
        ----------------------------------------------------------------
    




.. GENERATED FROM PYTHON SOURCE LINES 145-146

stop mapdl

.. GENERATED FROM PYTHON SOURCE LINES 146-147

.. code-block:: default

    mapdl.exit()








.. rst-class:: sphx-glr-timing

   **Total running time of the script:** ( 0 minutes  0.426 seconds)


.. _sphx_glr_download_vm-001-statically_indeterminate_reaction_force_analysis.py:

.. only:: html

  .. container:: sphx-glr-footer sphx-glr-footer-example


    .. container:: sphx-glr-download sphx-glr-download-python

      :download:`Download Python source code: vm-001-statically_indeterminate_reaction_force_analysis.py <vm-001-statically_indeterminate_reaction_force_analysis.py>`

    .. container:: sphx-glr-download sphx-glr-download-jupyter

      :download:`Download Jupyter notebook: vm-001-statically_indeterminate_reaction_force_analysis.ipynb <vm-001-statically_indeterminate_reaction_force_analysis.ipynb>`


.. only:: html

 .. rst-class:: sphx-glr-signature

    `Gallery generated by Sphinx-Gallery <https://sphinx-gallery.github.io>`_
