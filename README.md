# flavoprotein_db_sample_graph

Instructions to create your own graph:
(Note, this has only been tested in a Windows environment)

1. Install and set-up Neo4j community server edition 4.1 or newer, instructions to do that are here: https://neo4j.com/download-center/#community You will have to install JDK 11 (described on instruction page) and then edit your environmental variables to point "JAVA_HOME" to the directory you installed JDK 11 to. Then you should install Neo4j as a service (not console) with command "neo4j install-service" in the "bin" subfolder. For example  "C:\neo4j-community-4.1.3\bin neo4j install-service" in your command line. 
2. After installation, go to the neo4j browser. To do that, you will first have to start neo4j by navigating to the "bin" subfolder, for example: "C:\neo4j-community-4.1.3\bin neo4j start". You should then be able to use an internet browser (such as Google Chrome) to see the graph at "http://localhost:7474"
3. Save the data files to the neo4j "import" subfolder. 
4. Copy and paste the Cypher LOAD CSV Commands below one at a time, in order, to load the data from each file into the graph. The order is important because the "MERGE" command will match every property you pass it, so if you try to match based on pdb code and but not resolution when importing structural data it will create a new pdb node with no resolution even if a pdb structure already exists. Execute the last command for each subset (0 to 19).

LOAD CSV WITH HEADERS FROM "file:///PDBbind_Ki.csv" AS row
MERGE (e:Enzyme {enzymeName: row.Enzyme, EC_Number: row.Ecnumber})
MERGE (p:PDB {PDBCode: row.PDBCode,ecNumber: row.Ecnumber,Resolution: toFloat(row.Resolution)})
MERGE (l:Ligand {LigandID: row.Ligand})
MERGE (l)-[a:INHBITS {affinity: toFloat(row.Affinity_uM)}]->(p)
MERGE (e)-[s:HAS_STRUCTURE]->(p);

LOAD CSV WITH HEADERS FROM "file:///PDBbind_Kd.csv" AS row
MERGE (e:Enzyme {enzymeName: row.Enzyme, EC_Number: row.Ecnumber})
MERGE (p:PDB {PDBCode: row.PDBCode,ecNumber: row.Ecnumber,Resolution: toFloat(row.Resolution)})
MERGE (e)-[r:HAS_STRUCTURE]->(p)
MERGE (l:Ligand {LigandID: row.Ligand})
MERGE (p)-[r2:BINDS{affinity: toFloat(row.Affinity_uM)}]->(l);

LOAD CSV WITH HEADERS FROM "file:///KEGG_reduction_pots_added_all_known_10.24.20_edited2_enzymes_only.csv" AS row
MERGE (e:Enzyme {EC_Number: row.EC_Number})
ON CREATE SET e.enzymeName = row.Enzyme_Name
ON MATCH SET e.additionalEnzymeNames = row.Enzyme_Name
SET e.reductionPotential_mV = toFloat(row.Half_Reaction_RP_mV)
SET e.substratesKEGG = row.Substrates_KEGG_IDs
SET e.productsKEGG = row.Products_KEGG_IDs
SET e.substrates = row.Substrates_Names
SET e.products = row.Products_Names
SET e.rxnsKEGG = row.KEGG_RXN_ID;

:auto USING PERIODIC COMMIT 500
LOAD CSV WITH HEADERS FROM 'file:///HBPLUS_isoalloxazine_only_made_2020_10_16_lig_info_added_fixed_r_subset_0.csv' AS row
MERGE (p:PDB {PDBCode: row.pdbcode})
MERGE (f:Flavin {FlavID: row.FlavID, FlavType: row.FlavLigID})
MERGE (l:Ligand {LigandID: row.ligID, AminoAcid: row.amino_acid, WaterYN:row.H2O, NonAALig: row.ligand})
MERGE (a:FlavinAtom {AtomID: row.AtomID, AtomName:row.flav_atomID, FlavAtomName:row.FlavAtomName})
MERGE (n:NeighborAtom {AtomID: row.OtherAtomID, AtomName: row.lig_atomID, LigandID: row.ligID, AminoAcid: row.amino_acid, WaterYN: row.H2O, NonAALig: row.ligand, AtomType: row.atom, Charge: row.charge, Aromaticity: row.aromaticity, StereoChem: row.stereo_chem, SideChain: row.sidechain, Backbone: row.backbone})
MERGE (p)-[r:HAS_FLAVIN]->(f)
MERGE (p)-[b:BINDS]->(l)
MERGE (f)-[h:HAS_FLAVATOM]->(a)
MERGE (l)-[s:HAS_LIGATOM]->(n)
MERGE (a)-[t:HAS_NEIGHBOR {Distance:toFloat(row.DAdist), Angle: toFloat(row.DAAAangle)}]->(n);


References:
Protein Databank: H.M. Berman, J. Westbrook, Z. Feng, G. Gilliland, T.N. Bhat, H. Weissig, I.N. Shindyalov, P.E. Bourne. (2000) The Protein Data Bank Nucleic Acids Research, 28: 235-242. <rcsb.org>
HBPLUS: I.K. McDonald and J.M. Thornton (1994). Satisfying hydrogen bonding potential in proteins. J. Mol. Biol., 238, 777-793.
PDBbind: Wang R, Fang X, Lu Y, Yang CY, Wang S (2005). "The PDBbind database: methodologies and updates". J. Med. Chem. 48 (12): 4111–9. doi:10.1021/jm048957q. PMID 15943484. 
Kanehisa, M. and Goto, S.; KEGG: Kyoto Encyclopedia of Genes and Genomes. Nucleic Acids Res. 28, 27-30 (2000). 
