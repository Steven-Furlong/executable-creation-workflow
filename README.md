# Executable Creation Workflow

The following is a workflow guide into creating executable drivers for HID controllers (Arduino, Raspberry, etc.)





### Software Required



* iexpress - Build .exe bundle
* Resource Hacker - Change .exe icon
* Arduino IDE - Code encryption + export
* flash.bat script

###### 

##### flash.bat script

* Changes to be made in three locations as bolded within the script: board type x2 | .hex file name



@echo off

:: MANDATORY: This tells the script to look for avrdude in the IExpress temp folder

cd /d "%~dp0"

setlocal enabledelayedexpansion



echo === Button Box Auto-Updater ===

echo Searching for your device...



:: 1. Find the initial COM port

set "OLD\_PORT="

for /f "tokens=2 delims=()" %%a in ('wmic path Win32\_PnPEntity get caption /format:list ^| findstr /i "**Arduino Leonardo** USB-SERIAL"') do (

&nbsp;   set "fullport=%%a"

&nbsp;   set "OLD\_PORT=!fullport:COM=!"

)



if "%OLD\_PORT%"=="" (

&nbsp;   echo \[ERROR] No device found! 

&nbsp;   echo Ensure the box is plugged in and check Device Manager.

&nbsp;   pause

&nbsp;   exit

)



echo Found device on COM%OLD\_PORT%.

echo Resetting device into update mode...

:: Sends the 1200 baud signal to trigger the bootloader

mode COM%OLD\_PORT% BAUD=1200 > nul



:: 2. Wait for the computer to re-detect the chip on its NEW port

echo Waiting for Bootloader to appear...

timeout /t 4 > nul



:: 3. Re-scan to find the NEW COM port (the Bootloader port)

set "NEW\_PORT="

for /f "tokens=2 delims=()" %%a in ('wmic path Win32\_PnPEntity get caption /format:list ^| findstr /i "**Arduino Leonardo** USB-SERIAL Bootloader"') do (

&nbsp;   set "fullport=%%a"

&nbsp;   set "NEW\_PORT=!fullport:COM=!"

)



if "%NEW\_PORT%"=="" (

&nbsp;   echo \[ERROR] Could not find device after reset.

&nbsp;   echo Please try running the tool again or perform a manual double-tap reset.

&nbsp;   pause

&nbsp;   exit

)



echo Found Bootloader on COM%NEW\_PORT%.

echo Starting upload...



:: 4. Flash the NEW port using the embedded files

avrdude.exe -v -p atmega32u4 -c avr109 -P COM%NEW\_PORT% -b 57600 -D -U **flash:w:TSV1.0.hex:i**



if %ERRORLEVEL% NEQ 0 (

&nbsp;   echo.

&nbsp;   echo =================================================

&nbsp;   echo \[FAILED] The upload encountered an error.

&nbsp;   echo 1. Try a different USB port.

&nbsp;   echo 2. Try the manual "Double Tap" reset pins method.

&nbsp;   echo =================================================

) else (

&nbsp;   echo.

&nbsp;   echo =================================================

&nbsp;   echo SUCCESS! Your Button Box is ready.

&nbsp;   echo You can now unplug it and launch your sim.

&nbsp;   echo =================================================

)



pause





### Steps



1. Encrypt + export source code via Arduino IDE's "Export Compiled Binary"
2. Bundle the following into a folder for iexpress:

&nbsp;		- avrdude.conf

&nbsp;		- avrdude.exe

&nbsp;		- flash.bat

&nbsp;		- Encrypted code: Will now be in machine language format (.hex)



3\. The following iexpress settings should be used:

&nbsp;		- Extract files and run as an installation command

&nbsp;		- No prompt

&nbsp;		- Do not display a license.

&nbsp;		- Package the four files mentioned in step 2

&nbsp;		- Install program cmd /c flash.bat

&nbsp;		- Show window Default

&nbsp;		- Display message: 'Installation Complete' or similar

&nbsp;		- Check: Hide File Extraction Progress Animation from User (Cleaner)

&nbsp;		- No restart



4\. Use Resource Hacker to change .exe logo

&nbsp;		- Note that this cannot be a .jpg or .png image. Must be converted to .ico first.



5\. Upload .exe file to relevant channels (like a website's software page).





