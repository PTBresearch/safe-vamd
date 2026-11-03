
Overview of the framework
=========================

SAFE-VAMD is an open-source Python framework with high-level interfaces for computing outdoor sound propagation
based on vertical atmospheric modes. The framework has the following main capabilities:

-	Computing VA mode basis for range-independent, stratified atmospheres, using the SAFE method.

-	Projecting VA modes onto numerical and experimental pressure data to reconstruct outdoor sound fields
from sparse samples.

These main functionalities revolve around the SAFEComputation and the VAMDecomp classes, respectively, which are used
serially, as shown in Schematic 1.
The framework is highly modular and can be extended, modified, and combined. Schematic 1 shows the structure and
workflow, which is explained in the following sections. This section serves to familiarize the user with the recommended
workflow.

Computing VA modes with the SAFE-VAMD framework
===============================================

The steps for computing the VA modes with the framework are illustrated in Schematic 1 under the workflow
‘SAFE computation’ and are discussed in more detail as follows:

**Initialize a SAFE computation**. The user first initializes an instance of the *SAFEComputation* class.
The object contains all the necessary methods to compute the VA mode basis.

**Set up the SAFE computation**.  This step is executed by running the *setup* method.
At this stage, the user inputs a *.txt* file with semicolon-separated columns for height (in meters),
temperature (in :math:`K`), wind velocity (in :math:`ms^{-1}`), and relative humidity (in :math:`\%`).
The code configures a one-dimensional computational mesh with quadratic finite elements,
with nodes at the input heights. A PML with adjustable thickness is automatically
built at the top of the computational domain. The atmospheric profiles are fitted with Piece-Wise Cubic Hermite
Interpolating Polynomials (PCHIPs) by using the PchipInterpolator class from Python’s *SciPy* package (https://doi.org/10.1038/s41592-019-0686-2).
The air density is calculated using hydrostatic relations (see Eq. 9 in https://doi.org/10.1121/10.0002912),
the acceleration due to gravity, and the air density at ground level. Air pressure is computed with the ideal gas law.

**Compute the VA mode basis**. After the setup stage, the VA mode basis can be solved for by using the *solve* method.
A ground impedance condition (see https://doi.org/10.1121/10.0003567) can be enforced by the user at
this stage. As such, solutions for different ground impedances can be obtained without repeating the setup stage.
The method assembles the required matrices for the generalized eigenvalue problem (see Eq. 10 in https://doi.org/10.1016/j.enganabound.2025.106308).
and solves it by using the *linalg.eig* function in Python’s *NumPy* package (https://doi.org/10.1038/s41586-020-2649-2).

The computed VA modes are separated into the upwind and downwind mode basis, as discussed in Section 2.2 in https://doi.org/10.1016/j.enganabound.2025.106308.
After computing the VA mode basis, they can be exported to *.txt* file(s), with the *save_modes_in_txt* method.

Mode-decomposition with the SAFE-VAMD framework
===============================================

The SAFE-VAMD framework can be used to decompose a set of acoustic pressure samples onto VA modes.
This will result in complex-valued weights for each mode.
The weighted modes can then be superimposed to construct a sound field.
This can be achieved by following the workflow under “VAMD” in Schematic 1, further explained here:

**Initialize VAMD**. The *VAMDecomp* class contains all the functionality needed to decompose a set of acoustic pressure
samples onto the VA modes.

**Load the samples**. Load the acoustic pressure samples and their coordinates from a *.txt* file with semicolon-separated
columns for range (in :math:`m`), height (in :math:`m`), and the real and imaginary parts (both in :math:`Pa`
or dimensionless) of the acoustic pressure samples using the method *load_samples_from_txt*.

**Load the modes**. The computed upwind and downwind VA modes are loaded by using the method *load_modes_from_txt*.

**Apply a mode filter**.  The attribute *mode_filter* in *VAMDecomp* condenses the mode-filtering strategies in https://doi.org/10.1016/j.enganabound.2025.106308
into usable methods that the user can combine.
The effect of each mode filter can be checked by plotting the modal eigenvalues
in the real-complex plane with the method *mode_filter.plot_eigenvalues*.

**Match the modes to the samples**. This step solves the regression problem and computes
the complex-valued weights for the selected mode basis. The *linalg.pinv* function in the *SciPy* package (https://doi.org/10.1038/s41592-019-0686-2) is used
to solve the regression problem. The condition number of the regression problem, computed with the *linalg.cond* in the
*NumPy* package (https://doi.org/10.1038/s41586-020-2649-2), is optionally output.

**Export the modes**. The weighted upwind or downwind VA modes can be exported using the *save_modes_in_txt* method.
The resulting *.txt* file contains the modes' complex-valued weights.

Field reconstruction and post-processing
========================================

After matching the mode basis to the acoustic pressure samples, sound fields can be constructed by superimposing the
weighted modes.
To facilitate post-processing, we provide a high-level method (*get_plot_data*),
in the *AcousticField* class, which computes the acoustic pressure on a grid. The class also has methods to compute
the acoustic field at discrete points (*value_2d* and *value_2d_with_3d_spread*). Both the *SAFEComp* and *VAMDecomp*
classes have an instance of the *AcousticField* class as an attribute.

.. figure:: figures/documentation_framework_schematic.png
   :alt: Schematic 1
   :align: center

   Schematic 1 - Recommended workflow for the SAFE-VAMD framework.
