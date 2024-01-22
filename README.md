
# Fits Liberator installer on Windows
Explanation of progress in the development of the program installer with dates and descriptions of the errors solved.




## 09/01/2024 - The node modules used by the program were obsolete due to the last node update and creation of the program certificate.

- Resolved deprecaded node modules using a project folder with preloaded modules.
- To build the program, a folder `private` with the program certificate `fl4.pfx` is requested in the package.json, but this folder is not in the program's gui repository.




## 11/01/2024 - Creation of the program certificate and first attempt of the installer.

- The `private` folder was created within the gui and the program certificate was created using the command: 
```bash 
New-SelfSignedCertificate -DnsName noirlab.edu -Type CodeSigning -CertStoreLocation Cert:\CurrentUser\My
```
- Certificate was exported to `.pfx` file and copied to `private` folder inside the gui.
- It was possible to do a first `npm start` of the program.
- The program is running successfully if it is compiled from msys, but if it is compiled from the windows terminal the json files are not created for image analysis:
<div align="center">
    
![App Screenshot](https://github.com/Cristian-Infante/FL-on-Windows/blob/CFIC/image.png)
![App Screenshot](https://github.com/Cristian-Infante/FL-on-Windows/blob/CFIC/image2.png)

</div>




## 12/01/2024 – Final bug fixes and installer tests on different computers.

- The problem was found in the if statement at the line 294 from fitscli.cpp.
```c++
// On preloading we generate only png, and stats files
std::cout << "isPreloader: "<< ispreloader << endl;
if (ispreloader==1){
    int numimages,numplanes;
    string header;
    fits.getNumOfImagesPlanes(fptr,&numimages,&numplanes,&status,header);
    string histogramdata=histogram(preout,bins);
    std::cout << "outfilepath: "<< outfilepath << endl;

    saveheader(header,outfilepath);

    std::cout << "Flag" << endl;
    int has_undefined=0;
    if (nullvalues>0)
        has_undefined=1;
    if (stretch=="none")
        statsout=img.stats();
    else
        statsout=out.stats();
    double min = statsout(0, 0)[0];
    double max = statsout(1, 0)[0];
    double mean = statsout(4, 0)[0];
    double stdev = statsout(5, 0)[0];

    if (nullvalues>0){
        VipsOperations::correct_stats(nullvalues,npix,mean,stdev);
    }
    string histfile =  outfilepath + ".json";
    std::cout << "histfile: "<< histfile << endl;
    ofstream outhist(histfile);
    outhist << "{"<<'\"'<<"num_images"<<'\"'<<":" << numimages << ",\n";
    outhist << '\"'<<"num_planes"<<'\"'<<":" << numplanes << ",\n";
    outhist << '\"'<<"image"<<'\"'<<":" << ix << ",\n";
    outhist << '\"'<<"plane"<<'\"'<<":" << px << ",\n";
    outhist << '\"'<<"size"<<'\"'<<": {\n";
    outhist << '\"'<<"width"<<'\"'<<":" << preout.width() << ",\n";
    outhist << '\"'<<"height"<<'\"'<<":" << preout.height() << ",\n";
    outhist << '\"'<<"original_width"<<'\"'<<":" << original_width << ",\n";
    outhist << '\"'<<"original_height"<<'\"'<<":" << original_height << "\n";
    outhist << "}" << ",\n";
    outhist << '\"'<<"markpixel"<<'\"'<<": {\n";
    outhist << '\"'<<"undefined"<<'\"'<<":" << has_undefined << ",\n";
    outhist << '\"'<<"depthbit"<<'\"'<<":" << sizeofpixel << "\n";
    outhist << "}" << ",\n";
    outhist << '\"'<<"statistics"<<'\"'<<": {\n";
    outhist << '\"'<<"min"<<'\"'<<":" << min << ",\n";
    outhist << '\"'<<"max"<<'\"'<<":" << max << ",\n";
    outhist << '\"'<<"mean"<<'\"'<<":" << mean << ",\n";
    outhist << '\"'<<"stdev"<<'\"'<<":" << stdev << "\n";
    outhist << "}" << ",\n";
    outhist << '\"'<<"histogram"<<'\"'<<": [\n" << histogramdata << "]}";
}
```
- The problem was in the call of the saveheader function, the function was not returning correctly, but when the direct function code was placed directly where the function was called, the program worked without problems.
```c++
//saveheader(header,outfilepath);

// saveheader function code 
string headerfile = outfilepath + ".hdr";
std::cout << "headerfile: "<< headerfile << endl;
ofstream outheader(headerfile);
outheader << header << endl;
std::cout << "Flag1" << endl;
```
- A first build of the program was made. 
- When you run the program from another computer, a warning window appears saying the dlls needed for fitcli that are in `C:\msys64\ucrt64\bin`: `Libgcc_s_seh-1.dll Libcfitsio-4.dll Libgobject-2.0-0 .dll Libglib-2.0-0.dll Libstdc++-6.dll Libvips -cpp-42.dll Libvips-42.dll Libwinpthread-1.dll`.
- The solution to this problem was to create a zip of the `msys64` folder to copy it to the computer of the person who is going to install the program. This folder has the dlls necessary for the correct functioning of fitscli (It is necessary to add that folder to the path in the environment variables for the correct functioning of the program).
- It is necessary that the name of the computer user does not have spaces for the correct functioning of the program, (Example: `C:\Users\Cristian`) since the program will use the command `C:\Users\Cristian\AppData\Local\Programs\fitsliberator\fitscli\win\fits` and if you have spaces in the username (Example: `C:\Users\Cristian Infante\AppData\Local\Programs\fitsliberator\fitscli\win\fits`) you will encounter the following error depending on your default terminal: `"C:\Users\Cristian" is not recognized as an internal or external command, program or executable batch file`.
- Tests were carried out on different computers and if the recommendations given are followed (Copy the msys64 folder and have a username without spaces) both the installer and the program work correctly.



## 22/01/2024 - Creation of executable for msys64 folder

- To avoid errors in the path or malfunction of the program, an executable is created that is responsible for creating the msys64 folder in the necessary path and adding it to the environment variables.
- Tests were carried out on different devices and both executables work correctly as long as the recommendations given are followed (Have a username without spaces).



## Related

- The zip files for both the `private` and `msys64` folders and the program installer can be found in the following [Link](https://drive.google.com/drive/folders/1izsJnDk1ZxvlpBX4PqeRQ7MiUBmAnbUI?usp=sharing).
- In the case of the `msys64` folder, it is necessary to unzip it and copy it to the path `C:\msys64\ucrt64\bin` to add that folder to the path in the environment variables for the correct functioning of the program.
