
# Fits Liberator installer on Windows
Explanation of progress in the development of the program installer with dates and descriptions of the errors solved


### 09/01/2024 - The node modules used by the program were obsolete due to the last node update and creation of the program certificate.

- Resolved deprecaded node modules using a project folder with preloaded modules.


- To build the program, a folder ```private``` with the program certificate ```fl4.pfx``` is requested in the package.json, but this folder is not in the program's gui repository.

### 11/01/2024

- The ```private``` folder was created within the gui and the program certificate was created using the command: 
```bash 
New-SelfSignedCertificate -DnsName noirlab.edu -Type CodeSigning -CertStoreLocation Cert:\CurrentUser\My
```


- The certificate was exported to the private folder using the following steps:
    1. Open Windows Certificate Manager by running `certmgr.msc` in the Windows search box or in a Run command (Win + R).
    2. In Certificate Manager, navigate to "Personal" -> "Certificates".
    3. Locate the certificate, which should be named as per the `DnsName` specified in the creation command. It should be something related to "noirlab.edu".
    4. Right-click on the certificate -> "All Tasks" -> "Export...".
    5. Follow the export wizard. Ensure to select the option "Yes, export the private key".
    6. Set a password for the `.pfx` file when prompted, and remember this password as it is needed for configuring electron-builder.
    7. Choose a name and save the `.pfx` file in a secure location.

- Once the certificate has been exported to a `.pfx` file, the electron-builder configuration file is updated with the path to the `.pfx` file and the password set during the export. It is ensured that electron-builder has access to this location during the build process.


- The program is running successfully if it is compiled from msys, but if it is compiled from the windows terminal the json files are not created for image analysis:
<!-- ![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here) -->


### 12/01/2024
- The problem was found in the if statement at the line 294 from fitscli.cpp.
```c++
// On preloading we generate only png, and stats files
std::cout << "isPreloader: "<< ispreloader << endl;
if (ispreloader==1)
{
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


    if (nullvalues>0)
    {
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


- DLLs necessary for the fitscli that are in ```C:\msys64\ucrt64\bin```:
```
Libgcc_s_seh-1.dll
Libcfitsio-4.dll
Libgobject-2.0-0.dll
Libglib-2.0-0.dll
Libstdc++-6.dll`
Libvips-cpp-42.dll
Libvips-42.dll
Libwinpthread-1.dll
```


- The solution to this problem was to create a zip of the C:\msys64\ucrt64\bin folder to copy it to the computer of the person who is going to install the program. This folder has the dlls necessary for the correct functioning of fitscli (It is necessary to add that folder to the path in the environment variables for the correct functioning of the program).


- It is necessary that the name of the computer user does not have spaces for the correct functioning of the funpack, (example: C:\Users\Cristian) since the program will use the command C:\Users\Cristian\AppData\Local\Programs \fitsliberator\fitscli\win\funpack and if you have spaces in the username (Example: C:\Users\Cristian Infante\AppData\Local\Programs\fitsliberator\fitscli\win\funpack) you will encounter the following error depending on your default terminal: "C:\Users\Cristian" is not recognized as an internal or external command, program or executable batch file.


- Tests were carried out on different computers and if the recommendations given are followed (Copy the C:\msys64\ucrt64\bin folder and have a username without spaces) both the installer and the program work correctly.
