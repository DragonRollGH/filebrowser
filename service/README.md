# Run as a service
To start filebrowser automatically and silently on Windows 10, you can run filebrowser as a service.

1. Put these files in the same directory of "filebrowser.exe".
2. Make sure you can run filebrowser successfully.
3. Download "Resource Kits" from Microsoft. (https://www.microsoft.com/en-us/download/details.aspx?id=17657)
4. Edit the path to "Resource Kits" in [addService.bat](addService.bat), and run it as administrator.
5. Edit the path to "filebrowser.bat" in [configService.reg](configService.reg), and run it.
6. Find "filebroswer" in  Service.msc, change it to "Auto".
7. Done.
