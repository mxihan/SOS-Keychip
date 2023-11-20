<img src="https://github.com/UiharuKazari2008/SOS-Keychip/blob/main/.resources/Iona.png"/>

# Savior of Song "Iona" Hardware Keychip for Windows
Keychip emulator that handles game disk encryption and application lifecycle management for arcade cabinet or other applications

## Important Note!
This is NOT in ANY WAY compatible with a official ALLS/Nu keychip/preboot and is designed to work with a sudo-ALLS setup where sgpreboot does not exist and is specically designed to recreate the hardware key requirement to use the game. This is not designed to be high security and can be intercepted without much work.

## ToDo
* Bi-Directional serial port communication encryption
* Add special update password keystore for decrypting update files
* Better lifecycle management
* Segatools.ini keychip updates or direct integration to amdeamon
* Support to auto-relaunch application on death

## Use Cases
* Protection of a game/application where you are distributing images that should only be used by someone that has a physical keychip
* Protection of a game/application when the host in transport
* Prevention of offline data scraping

## Hardware
Waveshare RP2040-GEEK<br>
<img src="https://github.com/UiharuKazari2008/SOS-Keychip/blob/main/.resources/IMG_5948.jpg"/><br>
or any generic RP2040/[Arduino USB Dongle](https://a.co/d/fG1uoK3)

## Setup
0. Download the latest executables (and VHD Images if this is your first time)
  * https://github.com/UiharuKazari2008/SOS-Keychip/releases/download/release/savior_of_song_keychip.exe
  * https://github.com/UiharuKazari2008/SOS-Keychip/releases/tag/VHD-Templates
1. Create a device_key.h file in the ./Keychip-<version> folder
```cplusplus
const char* keychipText = "XXXX XX XX";
const char* keychipID = "XXXX-XXXXXXXXXXX";
const char* applicationID = "XXXX";
const char* applicationKey = "GAME_KEY";
const char* applicationIV = "EXPECTED_CLIENT_IV";
```
2. Launch Arduino IDE and flash the firmware
  * Install the following libraries with the library manager
    * ArduinoBearSSL
    * Adafruit_ST7789 (ONLY If your using the ST7789 version)
  * Version Explanation
    * ST7789 - Waveshare RP2040-GEEK
      * "Premium" Keychip with Display
    * HS-ST7735 - Waveshare RP2040-GEEK
      * High Security "Premium" Keychip with Display and OTA checkin
      * Flash should be encypted with esptool
    * RGB_INV - Pimoroni Tiny2040 or Generic RP2040
      * Standard Generic Version with (or without) a LED (Inverted Output)
    * ARDUINO - Generic Arduino Version
      * This generic and cheap bearbone version
3. Unzip release.zip in folder with the blank VHD to your new game folder
4. Mount and Encrypt the volumes<br>
**RUN AS ADMINISTRATOR**
```powershell
& ./savior_of_song_keychip.exe --ivString IV_STATIC_STRING_GOES_HERE --applicationID XXXX --applicationVHD app.vhd --optionVHD option.vhd --encryptSetup
```
5. Run the Keychip bootstrap<br>
**RUN AS ADMINISTRATOR**
  * Keychip should be on `COM5` or use `--port COM#` to change
```powershell
& ./savior_of_song_keychip.exe --ivString IV_STATIC_STRING_GOES_HERE --applicationID XXXX --applicationVHD app.vhd --appDataVHD appdata.vhd --optionVHD option.vhd
```
  * Game ID and IV String MUST MATCH device_key.h at all times!
  * Disks will be mounted to the following locations:
    * app -> X:\
    * appdata -> Y:\
    * option -> Z:\
    * **Please keep any existing volumes not mounted there!**
6. When Encryption is completed, and you have loaded your game data then checkout the keychip<br>
**RUN AS ADMINISTRATOR**
```powershell
& ./savior_of_song_keychip.exe --applicationVHD app.vhd --appDataVHD appdata.vhd --optionVHD option.vhd --shutdown
```

## Update Games Data
To update option data you must add `--updateMode` to enable read-write access
```powershell
& ./savior_of_song_keychip.exe --ivString IV_STATIC_STRING_GOES_HERE --applicationID XXXX --optionVHD option.vhd --updateMode
```
  * Remember to --shutdown or the hardware will lockout

## Proper Shutdown
If you are not restarting/powering off your hardware after the game, you must check-out otherwise the keychip will lockout. Run this command after game exe has close.
```powershell
& ./savior_of_song_keychip.exe --applicationVHD app.vhd --appDataVHD appdata.vhd --optionVHD option.vhd --shutdown
```

## Error Codes
The keychip is designed to handle requests in a very specific order and if any command is ran that is not at the correct stage the device will lock and require a power cycle.
* 0001 - Application does not match keychip's known store (Unlock Disk)
* 0090 - Invalid Check String (Unlock Disk)
* 0091 - Illegal unlock of disk that has previouly been unlocked
* 9000 - Unknown Error when unlocking disk
* 0013 - Generic Unlock Error
* 0010 - Tried to take ownership when the device is already in use
* 0011 - Tried to release ownership when the device was never in use
* 0013 - Requested Keychip ID before unlock of any disks

## Command Line Options
```powershell
      --help             Show help                                     [boolean]
      --port             Keychip Serial Port                            [string]
      --ivString         Challenge String                               [string]
      --applicationID    Game ID                                        [string]
      --applicationVHD   Application Disk Image                         [string]
      --appDataVHD       App Data Disk Image                            [string]
      --optionVHD        Options Disk Image                             [string]
      --env              Environment Configuration File                 [string]
      --secureEnv        Secure Environment Configuration File          [string]
      --launchApp        Run X:game.ps1 after check-in and handle check-out on
                         close
      --applicationExec  File to execute instead of game.ps1            [string]
      --prepareScript    PS1 Script to execute to prepare host          [string]
      --cleanupScript    PS1 Script to execute when shutting down       [string]
      --updateMode       Enable Update Mode for Volumes
      --shutdown         Shutdown Volumes (Check-Out)
      --encryptSetup     Setup Encryption of Volumes
      --dontCleanup      Do not unmount all disk images mounted
      --watchdog         Run as Watchdog to detect removal or failure
```

## Basic "just run the application" BAT file<br/>
**RUN AS ADMINISTRATOR**<br/>
This will login and launch one of the following (X:\game.ps1 or X:\bin\game.bat)
```powershell
savior_of_song_keychip.exe --ivString IV_STATIC_STRING_GOES_HERE --applicationID XXXX --applicationVHD app.vhd --optionVHD option.vhd --launchApp
```

## Creating your own sgpreboot (Fancy "this is a ALLS" setup)
**C:\ should be encrypted with TPM at all times and enable write filter for C:\ if required**<br/>

### System Folder
The system folder should be located in `C:\SEGA\system` and contain the following files:<br>
```
savior_of_song_keychip.exe
secure.ps1
```
#### secure.ps1
This is placed inside the system folder because it should be protected by BitLocker
```powershell
$game_id = "XXXX"
$game_iv = "EXPECTED_CLIENT_IV"
```
### Application Folder
This should be located in a decrypted partition mounted at `S:\XXXX` (where **XXXX** is your game id), it should cotain the following files:<br/>
```
    Directory: S:\XXXX


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        09/11/2023     19:25                preboot_data
d-----        09/11/2023     19:18                V0001
d-----        09/11/2023     19:18                V0002
-a----        09/11/2023     21:46            331 dismount.ps1
-a----        09/11/2023     21:25            632 enviorment.ps1
-a----        09/11/2023     21:45            222 mount-direct.ps1
-a----        09/11/2023     21:22            869 prepare.ps1
-a----        09/11/2023     21:22            869 cleanup.ps1
-a----        13/11/2023     23:03            844 start.ps1
-a----        10/11/2023     02:04           1182 stop.ps
```
Each game version should be in a separate folder and contain the VHDs<br/>
```
    Directory: S:\XXXX\V0001


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        13/11/2023     23:03     8615190528 app.vhd
-a----        13/11/2023     23:03       23130112 appdata.vhd
-a----        13/11/2023     23:03     3303301120 option.vhd
```
#### enviorment.ps1
The enviorment file is loaded before each start/stop/mount/dismount
```powershell
. C:\SEGA\system\secure.ps1

$game_folder = "S:\${game_id}"
$version = $(Get-Content -Path "${game_folder}\preboot_data\disk")
$audio_dev = "Multichannel Output*"

if ($version -eq 10) {
  $install = "${game_folder}\V0001"
} elseif ($version -eq 11) {
  $install = "${game_folder}\V0002"
}

$base = "${install}\app.vhd"
$data = "${install}\appdata.vhd"
$option = "${install}\option.vhd"
```
Create a if statement for each version you would want to "disk swap" with, set the version in the `preboot_data\disk` file (just a number or string)
#### prepare.ps1
This file should contain tasks like:
* Setting audio devices
* Rotate Display
* Set Display Refresh Rate
* Close Applications
* Enter Lockdown Mode
This is a example:
```powershell
Write-Host "ソフトウェアの設定 ." -NoNewline
Get-Process -Name slidershim -ErrorAction SilentlyContinue | Stop-Process -ErrorAction Stop
Write-Host "." -NoNewline
Get-Process | Where-Object {$_.MainWindowTitle -eq "Sequenzia - [InPrivate]"} | Stop-Process -ErrorAction SilentlyContinue
Write-Host "." -NoNewline
Stop-ScheduledTask -TaskName "StartHDMIAudio" -ErrorAction SilentlyContinue
Write-Host "." -NoNewline
taskkill /F /IM explorer.exe | Out-Null
Write-Host "." -NoNewline
if ($(Get-Process -Name mono-to-stereo -ErrorAction SilentlyContinue).Count -gt 0) {
    Get-Process -Name mono-to-stereo -ErrorAction SilentlyContinue | Stop-Process -ErrorAction SilentlyContinue
}
Write-Host ". [OK]"
```
#### start.ps1
This is what you call **as administrator** when you are starting the game
* This will look for a USB Drive called "SOS_INS" and install the latest option packs into Z:\
* Update must be 7z format and contain only option folders in the root of the archive
* Installation of Option packs will cause a full check-in and check-out of the keychip in update mode, Please be present at the cabinet and ensure the host is secure as the disks will be read-write during the updates!
* Later versions will support full installations based on the SOS VHD format and encrypted updates
```powershell
Write-Host "############################"
Write-Host " SOS app_boot"
Write-Host "############################"
. .\enviorment.ps1
if ((Get-Volume -FileSystemLabel SOS_INS -ErrorAction SilentlyContinue | Format-List).Length -gt 0) {
    $letter = (Get-Volume -FileSystemLabel SOS_INS).DriveLetter
    & C:\SEGA\system\savior_of_song_keychip.exe --ivString $game_iv --applicationID $game_id --applicationVHD $base --appDataVHD $data --optionVHD $option --updateMode
    Get-ChildItem -Path "${letter}:\*.7z" | ForEach-Object {
        & 'C:\Program Files\7-Zip\7z.exe' x -aoa -oZ:\ "${_}"
    }
    & C:\SEGA\system\savior_of_song_keychip.exe --applicationVHD $base --appDataVHD $data --optionVHD $option --shutdown
}
& C:\SEGA\system\savior_of_song_keychip.exe --ivString $game_iv --applicationID $game_id --applicationVHD $base --appDataVHD $data --optionVHD $option --prepareScript ".\prepare.ps1" --cleanupScript ".\shutdown.ps1" --launchApp
```
#### stop.ps1
Closes the application to allow the keychip to shutdown
```powershell
Get-Process -Name inject_x86 -ErrorAction SilentlyContinue | Stop-Process -ErrorAction SilentlyContinue
Get-Process -Name inject_x64 -ErrorAction SilentlyContinue | Stop-Process -ErrorAction SilentlyContinue
Get-Process -Name <GAME_EXE> -ErrorAction SilentlyContinue | Stop-Process -ErrorAction SilentlyContinue
```
`<GAME_EXE>` is the actual games process name, like *mercury*
#### cleanup.ps1
What is called to stop the game processes and clean up (needed if your automating and this is a multi-purpose system)
```powershell
. .\enviorment.ps1
if ((Get-Process -Name explorer).Length -eq 0) { & explorer.exe }
Start-ScheduledTask -TaskName "JVSDisable" -ErrorAction Stop
Stop-ScheduledTask -TaskName "StartALLSRuntime" -ErrorAction SilentlyContinue
Start-ScheduledTask -TaskName "EnableVNC" -ErrorAction SilentlyContinue
Get-AudioDevice -List | Where-Object { $_.Type -eq "Playback" -and $_.Name -like "VoiceMeeter Aux Input*" } | Set-AudioDevice | Out-Null
if (Test-Path "X:\") {
    & C:\SEGA\system\savior_of_song_keychip.exe --ivString $game_iv --applicationID $game_id --applicationVHD $base --appDataVHD $data --optionVHD $option --shutdown
}
```
#### mount-direct.ps1
Used to mount the game disks as update mode
```powershell
cd S:\XXXX\

. .\enviorment.ps1
. .\dismount.ps1
& C:\SEGA\system\savior_of_song_keychip.exe -v --ivString $game_iv --applicationID $game_id --applicationVHD $base --appDataVHD $data --optionVHD $option --updateMode
```
**Remember to dismount or you will lockout the keychip**
#### dismount.ps1
```powershell
cd S:\XXXX\

. .\enviorment.ps1
if (Test-Path "X:\") {
    & C:\SEGA\system\savior_of_song_keychip.exe --applicationVHD $base --appDataVHD $data --optionVHD $option --shutdown
}
Get-Disk -FriendlyName "Msft Virtual Disk" -ErrorAction SilentlyContinue | ForEach-Object { Dismount-VHD -DiskNumber $_.Number -Confirm:$false }
```

### Application VHD Filesystems
#### app.vhd
This should contain the base application and all thats required to run the application
```
    Directory: X:\


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        08/11/2023     15:46                bin
-a----        10/11/2023     02:04           1182 game.ps1


    Directory: X:\bin


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----l        08/11/2023     15:46                amfs
d----l        08/11/2023     15:46                appdata
d----l        08/11/2023     15:46                option
-a---l        08/11/2023     15:45              0 segatools.ini
```
This will be a Read-Only filesystem when active
* `amfs` links to Y:\amfs
* `appdata` links to Y:\appdata
* `options` links to Z:\
* `segatools.ini` links to Y:\segatools.ini
##### game.ps1
This is the script to launch the game. 
This is considered "secure" and is located in the game to prevent modification 
```powershell
. S:\XXXX\enviorment.ps1
$Mouse=@' 
[DllImport("user32.dll",CharSet=CharSet.Auto, CallingConvention=CallingConvention.StdCall)]
public static extern void mouse_event(long dwFlags, long dx, long dy, long cButtons, long dwExtraInfo);
'@ 
$SendMouseClick = Add-Type -memberDefinition $Mouse -name "Win32MouseEventNew" -namespace Win32Functions -passThru
Function Move-Mouse {

`	Param (
        [int]$X, [int]$y

    )

Process {

        Add-Type -AssemblyName System.Windows.Forms
        $screen = [System.Windows.Forms.SystemInformation]::VirtualScreen
        $screen | Get-Member -MemberType Property
        $screen.Width = $X
        $screen.Height = $y
        [Windows.Forms.Cursor]::Position = "$($screen.Width),$($screen.Height)"


    }
}
Move-Mouse -X 1920 -y 0 | Out-Null

$game_exec = "GAME_NAME.exe"
$game_hook = "GAME_HOOK.dll"
$am_hook = "AM_HOOK.dll"
$am_opts = "JSON FILES"

Write-Host "マルチチャンネルオーディオ構成のセットアップ..." -NoNewline
Get-AudioDevice -List | Where-Object { $_.Type -eq "Playback" -and $_.Name -like "${audio_dev}" } | Set-AudioDevice -ErrorAction Stop | Out-Null
Write-Host " [OK]"
Write-Host "JVS ハードウェアの初期化 ..." -NoNewline
Start-ScheduledTask -TaskName "JVSEnable" -ErrorAction Stop
Sleep -Seconds 3
Write-Host " [OK]"

Write-Host "############################"
cd X:\bin\
Get-Process -Name amdaemon -ErrorAction SilentlyContinue | Stop-Process -Force:$true -ErrorAction SilentlyContinue
Start-Process -WindowStyle Minimized -FilePath inject_x64.exe -ArgumentList "-d -k ${am_hook} amdaemon.exe -f -c ${am_opts}"
& .\inject_x86.exe -d -k ${game_hook} ${game_exec}
Write-Host ""
Write-Host "############################"
Write-Host " TERMINATED"
Write-Host "############################"
```
#### appdata.vhd
This is Read-Write storage for all app data and configuration files
```powershell

    Directory: Y:\


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        08/11/2023     15:43                amfs
d-----        08/11/2023     15:43                appdata
-a----        08/11/2023     15:43            379 segatools.ini
```
* You can move segatools.ini overtop the one in X:\bin if you want to prevent modification
#### option.vhd
Contains all option folders and is empty by default

## Build EXE
```powershell
pkg -t node18 --compress GZip .
npx resedit --in .\build\RP-KeychipEmulator.exe --out .\build\savior_of_song_keychip.exe --icon 1,iona.ico --no-grow --company-name "Academy City Research P.S.R." --file-description "I-401 Keychip" --product-version 1.5.0.0 --product-name 'Savior Of Song Keychip "Iona"'
```
