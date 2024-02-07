# JUMPsilactmt-Version-1.0.0
## Contents <br>
•	Introduction <br>
•	Release notes <br>
•	Software and Hardware Requirements <br>
•	Installation <br>
•	Input Files and their Preparation <br>
•	Update the parameter file <br>
•	Run the JUMPsilactmt program <br> 
•	Output files <br> 
•	Sample program for joining protein quantification data from two batches and performing a QC <br>
•	Parameter file for the sample program to join two batches <br> 
•	Maintainers <br>
•	Acknowledgments <br>
•	References <br>

## Introduction <br>
<div align="justify"> 
The JUMP (Jumbo Mass Spectrometry-based Proteomics) search engine revolutionized peptide and protein identification by amalgamating pattern-based scoring and de novo tag scoring, significantly elevating the precision of peptide-spectrum matches (PSMs), as showcased in the study by Wang et al. (2014). Besides, the JUMP filtering engine conducted a comprehensive analysis of database search results, aiming to enhance the precision of peptide and protein identification, with insights from the works of Peng et al. (2003) and Xu et al. (2009). In the context of multi-batch TMT analysis, the JUMP batch program effectively addressed challenges related to systematic non-biological batch variations and the exclusive identification of specific peptides and proteins in certain batches.<br><br>
JUMPsilactmt integrated data inputs from the JUMP batch program and the mzXML files of individual samples within each batch. It encompassed a two-fold quantification approach involving MS1 level quantification/normalization (specifically SILAC quantification) and MS2 level quantification. The workflow started with extracting TMT reporter ion peaks and consolidating m/z shifts among these reporters. Furthermore, it utilized an additional round of TMT reporter ion peak extraction in the initial phases of MS2 level quantification. The process assigned the lowest intensity across the entire batch to the respective PSMs if the reporters lacked quantification. Additionally, the process applied a TMT impurity correction, assigning a value of 1 to reporters with an intensity of <=0 and subsequently incorporating the relevant proteins and peptides into the PSMs. Notably, this TMT impurity correction deviated slightly from the standard impurity correction used in our JUMP-q pipeline (Niu et al., 2017). In this case, the approach shifted from the previous evaluation of the final PSM quantification based on a maximum of (original reporter intensity/2) and impurity-corrected intensity to utilizing the impurity-corrected reporter intensities. The subsequent stages of MS2-level quantification included the application of loading bias and TMT noise corrections. Following the approach of Niu et al. (2017), the loading bias correction aimed to eliminate systematic sample processing discrepancies by normalizing reporter intensities for individual Lys PSMs. It involved the division of these intensities by PSM-wise mean values and their subsequent conversion into log-scale values. Consequently, the majority of PSM intensities centered around zero across the reporters. The trimmed mean intensity for each reporter was generated by excluding the top and bottom 10% of values, providing the necessary normalization factors that were then converted to raw-scale single Lys PSMs. The process excluded PSMs containing decoys, contaminants, or anomalies.<br><br>
The TMT noise correction process commenced with determining the minimum intensity observed across the entire batch. It is initiated by identifying noise channels for both light PSMs (representing the fully labeled channel) and heavy PSMs (representing the fully unlabeled channel). Subsequently, it set the TMT noise level for each PSM based on the first and second lowest signals. After this determination, it subjected both heavy and light PSMs to noise correction at varying levels as dictated by the TMT noise. Each light and heavy PSM underwent normalization based on the respective total intensity, and heavy PSMs were aggregated at the peptide and protein levels. The normalization involved the MS1 intensity ratio. This method encompassed the calculation of heavy and light peptide isotopic distributions for all identified peptides, including extracting heavy and light peptide MS1 intensities from each scan, focusing on the most significant peaks. It computed heavy and light peptide MS1 precursor intensities ratios for each PSM. Subsequently, it summarized the MS1 intensity ratio at the peptide level and normalized TMT noise corrected light PSMs using the corresponding MS1 L/H ratio. After summarizing TMT noise-corrected light and heavy PSMs into the corresponding peptides, it calculated each peptide's light-Lys% (L/(L+H)) using heavy and light peptides. Finally, it identified and removed outliers using the extreme studentized deviation (ESD) and Dixson Q-test methods. The entire pipeline concluded with the ultimate protein summarization.<br><br>
</div>
 
<div align="justify"> 
In additiion to protein quantification, JUMPsilactmt was vital when an experimental measurement of the free lysine pSILAC ratio was unavailable. However, among all identified peptides, there were tiny portions of the peptides that contained two Lys residues (i.e., double-K-peptides). The double-K-peptides should have three peaks (light, mixed, and heavy). The light peak may also be generated from pre-existing light proteins. <br><br>
</div>

If each protein ($P$) had a synthesis rate ($S$) following zero-order kinetics, and $H_A$% was the averaged percentage of heavy K, considering time $t$, we can derive the following: <br>

 
Total $P_M$ (mixed) $=$ $S \times t  \times$  $H_A$% $\times$ $(1-H_A$% $) \times 2$ (as both Lys had an equal chance to be heavy) <br>
Total $P_H$ (heavy) $= S \times t \times H_A$% $\times$ $H_A$% <br>
The ratio of the mixed to the heavy peptide ${ R (=  \frac{P_M}{P_H}  ) =   \frac{(1-H_A%) \times 2}{H_A%}}$ was independent of synthesis rates. <br>
Thus,
                                         HA% = 2/((2+R))                                 
                                         LA% = 1-HA%
We can use the above simple equations to derive the averaged LA% during the pulse (e.g., eight days) based on double-K-peptides. As the L% is dynamic, the L% on day 8 was not equal to the averaged LA% in 8 days. We should be able to fit the averaged LA% in 8 days to derive L% on day 8. 
The free lysine estimation process was like the quantification process. Unlike the quantification process, we derived double Lys PSM during the loading bias correction instead of single Lys PSM. Moreover, we inspected mixed and heavy PSMs in the TMT noise correction step and summarized them only to peptide level. The MS1 level quantification to peptide level and the mathematical procedure helped get the free lysine pSILAC ratio. 




## Release notes (Version 2.0.0) <br>
In the current version 
1. We assume that the mice's overall amount of proteins is unchanged over time as the mice used in this study are adults. 
2. The Lys used in protein synthesis originates from the food intake, with the rest recycled through protein degradation. 
3. Concentration of soluble Lys is conserved; the rate of free Lys absorbed from food is assumed to equal the rate excreted. 
4. Three different settings, which represent simple to comprehensive inputs. Here, setting 2 is the default in the current version.
5. Here, the default optimization algorithm is nonlinear least-squares.
5. The current version supports different time point inputs, such as 3, 4, and 5 point inputs, including 0 days.
6. This version can also calculate apparent half-lives.

## Software and Hardware Requirements <br>
The program was written in MATLAB language. The program runs on any Linux, Mac, or Windows computer with MATLAB R2014 (The MathWorks, Inc., Natick, Massachusetts, United States) or the above version. The current JUMPt program has been successfully tested with MATLAB R2021 version on the system: 16 GB memory and 3.3 GHz CPU processors with six cores. The program needs more time to complete on the system with fewer core processors in the CPU.
MATLAB toolbox needed: 
- Global Optimization toolbox along with other basic toolboxes

## Installation <br>
Installation of the script is not required. Download all the files/folders to any working directory (e.g., /home/usr/JUMPt) as shown in Figure 1. 

![Figure1](https://github.com/abhijitju06/JUMPt/assets/34911992/d5ff202e-c1a7-4d23-bb80-151f26b1028b)
<p align="center">
Figure 1
</p>

## Input File Preparation <br>
A testing dataset with 100 proteins is available for each setting. Besides, the testing dataset for different time points is also available for default setting 2. Like the testing dataset, the user must prepare the input data file with the information below.
•	pSILAC proteins (mandatory)
•	pSILAC free (unbound) Lys (optional)
•	Free Lys concentration (required, as setting 2 is the default in the current version)
•	Lys concentration in individual proteins (optional)

## Update the parameter file <br>
The JUMPt program requires a parameter file (JUMPt.parms). The user must specify the following parameters in the 'JUMPt.params' file.
•	JUMPt setting 
•	Input file name (along with the exact path)
•	Bin size('bin_size') to fit proteins each time 
•	MATLAB optimization algorithm
•	Number of time points
•	Purity of SILAC food 
•	Whether the user wants to calculate the apparent half-life

## Run the JUMPt program (Demo data set) <br>
Launch the MATLAB software and open the JUMPt main program file "Run_Main_File.m" in it, as shown in Figure 2. Press the "Run' button, as shown in the figure, to start the program. Once the program begins, it will show the progress of protein fitting and the successful completion (Figure 3).

![Figure2](https://github.com/abhijitju06/JUMPt/assets/34911992/8a4aeaf1-d008-4a23-a0e6-6ef08033c1ca)
<p align="center">
Figure 2
</p>

Nonlinear fitting of proteins and Lys data using ODE is computationally expensive, especially when the protein data is enormous (e.g.,> 1000 proteins). We divide the proteins into sets with bin sizes between 10-100 to reduce the computational complexity. The program finds the optimal degradation rates (turnover rates or half-lives) by fitting protein data (in setting-1) and free-Lys data (in setting-2 and setting-3).

![Figure3](https://github.com/abhijitju06/JUMPt/assets/34911992/d2bf25a5-32e6-43e0-a8fd-8117dd000fa0)
<p align="center">
Figure 3
</p>

## Output file information <br>
Two output Excel files are generated with the prefix 'results_Corrected_T50' and 'results_Apparent_T50' to the input file name. They will be rendered in the same folder where the input file is located. The results with proteins' corrected half-lives (in days) and confidence intervals will be saved to the output file. In addition, parameters used to calculate the half-lives will also be kept in the output file. Besides, the program will save the apparent half-lives in the output file.

## Maintainers <br>
To submit bug reports and feature suggestions, please contact:
Surendhar Reddy Chepyala (surendharreddy.chepyala@stjude.org), Junmin Peng (junmin.peng@stjude.org), Abhijit Dasgupta (abhijit.dasgupta@stjude.org), and Jay Yarbro (jay.yarbro@stjude.org). 

## Acknowledgment <br>
We acknowledge St. Jude Children's Research Hospital, ALSAC (American Lebanese Syrian Associated Charities), and the National Institute of Health for supporting the development of the JUMP Software Suite.

## References <br>
1. Chepyala et al., JUMPt: Comprehensive protein turnover modeling of in vivo pulse SILAC data by ordinary differential equations. Analytical chemistry (under review)
2. Wang, X., et al., JUMP: a tag-based database search tool for peptide identification with high sensitivity and accuracy. Molecular & Cellular Proteomics, 2014. 13(12): p. 3663-3673.
3. Wang, X., et al., JUMPm: A Tool for Large-Scale Identification of Metabolites in Untargeted Metabolomics. Metabolites, 2020. 10(5): p. 190.
4. Li, Y., et al., JUMPg: an integrative proteogenomics pipeline identifying unannotated proteins in human brain and cancer cells. Journal of proteome research, 2016. 15(7): p. 2309-2320.
5. Tan, H., et al., Integrative proteomics and phosphoproteomics profiling reveals dynamic signaling networks and bioenergetics pathways underlying T cell activation. Immunity, 2017. 46(3): p. 488-503.
6. Peng, J., et al., Evaluation of multidimensional chromatography coupled with tandem mass spectrometry (LC/LC-MS/MS) for large-scale protein analysis: the yeast proteome. Journal of proteome research, 2003. 2(1): p. 43-50.
7. Niu, M., et al., Extensive peptide fractionation and y 1 ion-based interference detection method for enabling accurate quantification by isobaric labeling and mass spectrometry. Analytical chemistry, 2017. 89(5): p. 2956-2963.
