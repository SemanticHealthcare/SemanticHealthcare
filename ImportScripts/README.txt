To correctly use the Semantic Healthcare information import scripts, you will need to download the OSEHRA VistA repository:
  https://github.com/OSEHRA/VistA

That repository contains the necessary code to interact with the VistA instance automatically.  To integrate the import scripts
with the VistA repository, move the ImportScripts folder to the VistA/Testing/ directory and add the following line to the end of
the CMakeLists.txt file that is found in the same directory.

  add_subdirectory(ImportScripts, SHImportScripts)
  
Then go to the corresponding directory in the build tree, VistA-build/Testing/SHImportScripts/, there should be three new python files 
that we will use for import.  

To run the import, you need to run two of the files that are found in that directory:
First
  $ python importPats.py 
Second
  $ python semanticDataImport.py 