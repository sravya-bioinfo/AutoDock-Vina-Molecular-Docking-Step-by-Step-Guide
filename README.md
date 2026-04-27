What is AutoDock Vina?
AutoDock Vina is an improved docking engine over AutoDock 4. It is faster, more accurate, and easier to use — no need to calculate grid maps separately. It uses a scoring function based on hydrophobic interactions and hydrogen bonding.
Key advantages over AutoDock 4:

Up to 100x faster
No AutoGrid step needed
Simpler configuration file
Better accuracy on average


Software Required
SoftwarePurposeDownloadAutoDock VinaDocking enginevina.scripps.eduAutoDockTools (ADT)Receptor & ligand preparationMGLToolsPyMOLVisualizationpymol.orgOpen BabelFile format conversionopenbabel.org

Workflow Overview
Protein (PDB) --> Remove Water & Heteroatoms --> Add Hydrogens & Charges --> receptor.pdbqt
Ligand (SDF/MOL2) --> Add Hydrogens & Torsions --> ligand.pdbqt
         |
    Define Grid Box (center x, y, z + size)
         |
    Write config.txt
         |
    Run AutoDock Vina
         |
    Analyze output.pdbqt + log
         |
    Visualize in PyMOL

Step 1 — Download the Protein Structure

Go to RCSB Protein Data Bank
Search your target protein (e.g., AKT1, 1IEP, 6LU7)
Download the .pdb file


Tip: Note the active site residues from the literature before starting — you'll need them to center the grid box.


Step 2 — Prepare the Receptor
Open AutoDockTools (ADT):
Steps in ADT:

File > Read Molecule → load your .pdb file
Edit > Delete Water → remove all water molecules
Edit > Delete > Delete Atom Type > HETATM → remove co-crystallized ligands (if any)
Edit > Hydrogens > Add → select Polar Only
Edit > Charges > Compute Gasteiger
Grid > Macromolecule > Choose → select your protein
Grid > Macromolecule > Save → saves as receptor.pdbqt

Alternatively, use command line with MGLTools Python scripts:
bashpythonsh prepare_receptor4.py -r protein.pdb -o receptor.pdbqt -A hydrogens

Step 3 — Prepare the Ligand
Get your ligand:

Download from PubChem in SDF format
Convert to .pdb using Open Babel if needed:

bashobabel ligand.sdf -O ligand.pdb --gen3d
In ADT:

Ligand > Input > Open → load ligand file
Ligand > Torsion Tree > Detect Root
Ligand > Torsion Tree > Choose Torsions → confirm rotatable bonds
Ligand > Output > Save as PDBQT

Or via command line:
bashpythonsh prepare_ligand4.py -l ligand.pdb -o ligand.pdbqt

Step 4 — Define the Grid Box
Unlike AutoDock 4, Vina does not need AutoGrid. You just define the box center and size directly in the config file.
In ADT:

Grid > Grid Box
Center the box on your active site
Note down:

Center: x, y, z coordinates
Size: size_x, size_y, size_z (in Angstroms, typically 20–30 Å)




Tip: Use PyMOL to identify and measure the active site dimensions for accurate box sizing.


Step 5 — Create the Configuration File
Create a plain text file named config.txt:
receptor = receptor.pdbqt
ligand = ligand.pdbqt

center_x = 12.345
center_y = -5.678
center_z = 23.456

size_x = 25
size_y = 25
size_z = 25

exhaustiveness = 8
num_modes = 9
energy_range = 4

out = output.pdbqt
log = log.txt
Parameter explanation:
ParameterDescriptionexhaustivenessSearch thoroughness (8 = default, increase for better results)num_modesNumber of binding poses to generateenergy_rangeMax energy difference from best pose (kcal/mol)

Step 6 — Run AutoDock Vina
Open terminal/command prompt in the folder containing all files:
bashvina --config config.txt
Or specify directly without config file:
bashvina --receptor receptor.pdbqt \
     --ligand ligand.pdbqt \
     --center_x 12.345 --center_y -5.678 --center_z 23.456 \
     --size_x 25 --size_y 25 --size_z 25 \
     --out output.pdbqt --log log.txt

This generates output.pdbqt (all docked poses) and log.txt (binding energies).


Step 7 — Analyze Results
Open log.txt — it shows binding affinities for each pose:
-----+------------+----------+----------
mode | affinity   | dist from best mode
     | (kcal/mol) | rmsd l.b.| rmsd u.b.
-----+------------+----------+----------
   1 |      -8.5  |    0.000 |    0.000
   2 |      -8.1  |    1.823 |    2.451
   3 |      -7.9  |    2.134 |    3.012
Best pose = Mode 1 (lowest binding affinity)
Key values:
ParameterMeaningAffinity (kcal/mol)Lower (more negative) = stronger bindingRMSD l.b.Lower bound RMSD from best modeRMSD u.b.Upper bound RMSD from best mode

Step 8 — Extract Best Pose
Split the output file to get individual poses:
bashvina_split --input output.pdbqt
This creates output_ligand_01.pdbqt, output_ligand_02.pdbqt, etc.
Mode 1 (_01) is your best docking pose.

Step 9 — Visualize in PyMOL
bashpymol receptor.pdb output_ligand_01.pdbqt
In PyMOL:

Show > Sticks for ligand
Show > Surface or Cartoon for receptor
Action > Find > Polar Contacts → show hydrogen bonds
Color by element for clarity

To show binding pocket residues only:
select binding_site, receptor within 5 of ligand
show sticks, binding_site

Sample Results Table
LigandBinding Affinity (kcal/mol)RMSD l.b.RMSD u.b.Compound A-9.20.0000.000Compound B-8.51.6542.341Compound C-7.82.012 .3.156

Lower binding affinity = stronger predicted binding interaction


AutoDock 4 vs AutoDock Vina — Quick Comparison
FeatureAutoDock 4AutoDock VinaSpeedSlowerUp to 100x fasterGrid mapsRequired (AutoGrid)Not requiredConfigurationComplex (.gpf, .dpf)Simple (config.txt)AlgorithmLGAIterated local searchAccuracyGoodGenerally betterBest forDetailed studiesScreening & beginners

Common Errors & Fixes
ErrorFix"Could not load receptor"Check .pdbqt format — ensure charges were added"Ligand outside the box"Re-center grid box or increase box sizeAll modes same energyIncrease exhaustiveness"No output generated"Check config.txt for typos in file namesPyMOL won't open .pdbqtConvert to .pdb using Open Babel first

Tips for Better Results

Use exhaustiveness = 16 or 32 for research-grade results (slower but more thorough)
Always remove water and heteroatoms from the receptor before docking
Validate your docking by re-docking the co-crystallized ligand and checking RMSD < 2 Å
Cross-check results with KEGG and STRING for pathway relevance


References

Trott & Olson (2010). AutoDock Vina. J Comput Chem, 31(2), 455–461
AutoDock Vina Manual
MGLTools Documentation
Open Babel Documentation


Author
Mulpuri Sravya
B.Tech Bioinformatics, VFSTR
LinkedIn · ORCID


This guide is based on hands-on academic experience in computational drug discovery, including virtual screening and molecular docking projects targeting cancer-related proteins.
