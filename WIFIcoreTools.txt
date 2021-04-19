<# Wifi Profile Management
.Synopsis
   This function group provides a set of tools to assist with the maintenance of a healthy WiFi profile list.
   It also assist with the management and core activities associated with WiFi profiles.
  General thanks To:
  https://devblogs.microsoft.com/scripting/using-powershell-to-view-and-remove-wireless-profiles-in-windows-10-part-4/
.Description
   Get-WifiProfile - gets the list of Wifi Profiles currently stored on this computer.
   Remove-WifiProfile - removes a named profile from the local machine.
   Make-WiFiProfilesBaseline - Takes the current profiles and treats that as the core set for future baselining 
         of for selecting preferences to connect with.
   Baseline-WiFiProfiles - Indicates profiles that in excess of the Baslined core profiles and allows you to 
         choose to remove profiles you may have added over time.
   Get-SSIDProfileInfo - Provides detailed information of the characteristics of current WIFI profiles
   Get-WiFiCoreBroadcasting - gets the first Core wifi profile that is currently within broadcasting range  
#>
<#
Author :  Robert Hindle
Created:  V1 2021-04-18
Limited Liability Statement: Use at your own discretion and risk.
The packaging is mine.  Some of the core functioning code is taken from other internet sites and 
discussion streams which have solved the steps of the process.  Thank you. 
#>
<# Get-WifiProfile
.Synopsis
   Gets and lists the names of WiFi Profiles currently found on this machine for this user
   Thanks for ideas how to do this To:
   https://devblogs.microsoft.com/scripting/using-powershell-to-view-and-remove-wireless-profiles-in-windows-10-part-4/
   https://devblogs.microsoft.com/scripting/get-wireless-network-ssid-and-password-with-powershell/
.Description
   Gets and lists names of Wifi Profiles found on this machine
.Parameter
    Name - Can specify 1 to N names to find.  No value retrieves All Profiles available
.Parameter
.Example
   Get-WifiProfile
.Example
   Get-WifiProfile -Name "MyHomeWIFI"
#>
Function Get-WifiProfile  {
   Param ($Name)
    $list=((netsh.exe wlan show profiles) -match '\s{2,}:\s') -replace '.*:\s' , ''

   if ($Name.length -le 0) { # No Names
      $list
   }
   else { # 1+ names
      Foreach ($Profile in $Name) {
         if ($Profile -in $Name) { $Profile }
      } 
   }
} # Get-WifiProfile

<# Remove-WifiProfile
.Synopsis
  Removes a named WiFi Profile
  Thanks for how to doe this To:
  https://devblogs.microsoft.com/scripting/using-powershell-to-view-and-remove-wireless-profiles-in-windows-10-part-4/
.Description
  Given a WiFi profile name it finds it and the removes it is found and reports that or that it was not found
.Parameter
  Name Name of the profile, in quotes if more than one word,
.Example
  Remove-WiFiProfile -Name "ForeignCoffeeShop"
#>
Function Remove-WifiProfile
{
     [cmdletbinding()]
     Param  ( [System.Array]$Name=$NULL
            )
     begin{}
     process 
     {
          Foreach ($item in $Name)
          {
          $Result=(netsh.exe wlan delete profile $item)

          If ($Result -match 'deleted')
            {
               "WifiProfile : $Item Deleted"
            }
          else
            {
               "WifiProfile : $Item NotFound"
            }
          }
      }
      end{}
} #Function

<# Make-WiFiProfilesBaseline
.Synopsis
   Makes a baseline list of Profiles that can be editted manually using NotePad
.Description
   Takes the current WiFi profile list on a machine and makes it the baseline list.
   Run a second or nth time list is updated to the new current machine list.
.Parameter
   BasePath - Path where WIFIcore.txt file is to exist.  Absolute path reccommended.
.Example
    Make-WiFiProfilesBaseline  
#>
Function Make-WiFiProfilesBaseline {
    Param ( $BasePath = "$PSScriptRoot\WIFIcore.txt")
    $current = Get-WifiProfile
    $current > $BasePath
    } # Make-WiFiProfilesBaseline

<# Baseline-WiFiProfiles
.Synopsis
   Goes through a machines current WiFi profiles and if they are not in the baseline list flags or deletes
.Description
   Compares current WiFi profiles with desired base Wifi profiles.
   if DebugWI is Yes then it flags the extra profiles.
   if DebugWI is NO then it uses netShell to remove the WiFi profiles.
.Parameter
   BasePath - path to get to list of expected WiFi profiles to be maintained
.Parameter
   Action - What-If option for debug and checks  Dflt=YES should be NO to physically delete
.Example
   Baseline-WiFiProfiles -BasePath "c:\temp\coreprofiles.txt" -DebugWI "NO"
#>
Function Baseline-WiFiProfiles {
    Param ( $BasePath = "$PSScriptRoot\WIFIcore.txt",
            $Action = "NO" )

 $base = Get-Content -Path $BasePath
 $current = Get-WifiProfile

 Foreach ($prof in $current) {
   $prof
   if ($prof -in $base) { }
   else {
      if ($Action -eq "YES") { 
         Remove-WifiProfile -Name "$($prof)"
      }
      else {
         "$prof should Delete"
      } # remove of inform
   } # Not in Base
 } # ForEach Profile
} # Baseline-WiFiProfiles

<# Get-SSIDProfileInfo
.Synopsis
   Given a name of a profile it returns the SSID and the access Key
.Description
    Given a name of a profile it returns the SSID and the access Key
.Parameter
    ProfileName Deflt="MyHomeProfile"
.Example
   Get-SSIDProfileInfo
.Example
   Get-SSIDProfileInfo -Key NO
#>
Function Get-SSIDProfileInfo {
  Param ( $ProfileName,
          $Key = "NO" )
  #$profilename
  if ($ProfileName.length -le 0) { $ProfileName = Get-WiFiProfile }
  foreach ($profile in $ProfileName){
    if ($Key -eq "YES") {
     $output = netsh.exe wlan show profiles name="$Profile" key=clear
    }
    else {
     $output = netsh.exe wlan show profiles name="$Profile"
    }
    #$output
    $SSIDsrchRslt = $output |Select-String -Pattern "SSID Name"
    $SSID = ($SSIDsrchRslt -split ":")[-1].Trim() -replace '"'
    $AUTHsrchRslt = $output |Select-String -Pattern "Authentication"
    $AUTH = ($AUTHsrchRslt -split ":")[-1].Trim()  
    $CONNsrchRslt = $output |Select-String -Pattern "Connection mode"
    $CONN = ($CONNsrchRslt -split ":")[-1].Trim()
    $PSWDsrchRslt = $output |Select-String -Pattern "Key"
    $PSWD = ($PSWDsrchRslt -split ":")[-1].Trim() -replace '"'
    "$($SSID.padright(15," ")) $($AUTH.padright(15," ")) $($CONN.padright(22," ")) $PSWD"
  }
}

<# Get-WiFiCoreBroadcasting
.Synopsis
   Find a broadcasting WIFI SSID that is specified as one of your core managed accounts
.Description
    Given a name of a profile it returns the SSID and the access Key.
    If none it tells you.  If 1 if names it.  If 2+ then it names the first one found.
.Parameter
    BasePath = File path to your baselined WIFI connections
.Example
   Get-WiFICoreBroadcasting
#>
Function Get-WiFiCoreBroadcasting {
   param (  $BasePath = "$PSScriptRoot\WIFIcore.txt"  )

          $BaseNames = Get-Content -Path $BasePath
          $BaseNameSet = "\s("
          Foreach ($BaseName in $BaseNames){
             $BaseNameSet += $BaseName
             $BaseNameSet += "|"
          }
          $BaseNameSet += ")\s"
          $BaseNameSet = $BaseNameSet.replace("|)",")")
          $regex0 = [regex]$BaseNameSet 

          $BroadNames = & netsh wlan show networks
          $M0 = $regex0.Matches($BroadNames)
          if ($m0.count -eq 1) {
           $M0.Value
          }
          else { # Not just 1 found
            if ($M0.count -lt 1) { "No Core Found" }
            else { 
               $($M0[0].Value)
            } # more than 1 so pick first
          }
} # Get-Wifi core Broadcasting

if ($WIFImgmtTEST -eq 10) {
   Get-WifiProfile
   }
   elseif ($WIFImgmtTEST -eq 50) {
   Remove-WifiProfile -Name "RA-Guest"
   }
   elseif ($WIFImgmtTEST -eq 100) {
   Make-WiFiProfilesBaseline
   }
   elseif ($WIFImgmtTEST -eq 150) {
   Baseline-WiFiProfiles
   }
   elseif ($WIFImgmtTEST -eq 200) {
   $list = Get-WifiProfile
   $machine = [environment]::MachineName
   "WiFi Profiles on $Machine"
   "SSID            Authentication  Connection          Key"
   foreach ($prof in $list) {
      Get-SSIDProfileInfo -ProfileName $prof.name
      }
   "WiFi Profiles on $Machine"
   "SSID            Authentication  Connection          Key"
   Get-SSIDProfileInfo -Key "NO"
   "WiFi Profiles on $Machine"
   "SSID            Authentication  Connection          Key"
   Get-SSIDProfileInfo -ProfileName "Sanctuary"
   }
   elseif ($WIFImgmtTEST -eq 250) {
     Get-WiFiCoreBroadcasting
   }
   ####### MAIN #######
$Stoppers = @("exit","stop","end","halt","enough")
Do {
#Display the Options
 " "
 " "
 " WIFI Core Tools - V1 "
   "1) Get-WifiProfile - gets the current list of Wifi Profiles"
   "     Arg = Profile Name"
   "2) Remove-WifiProfile - removes a named profile"
   "     Arg = Profile Name"
   "3) Make-WiFiProfilesBaseline - Makes current profiles as core Baselined profiles."
   "     Arg = Base path. Default is local Wificore.txt" 
   "4) Baseline-WiFiProfiles - Compairs Current profiles to Baselined profiles. Helps remove extras."
   "     Arg = Action  = YES or NO. Default is NO (just inform)."
   "5) Get-SSIDProfileInfo - Provides detailed information on current WIFI profiles."
   "     Arg = Show Keys = YES or NO.  Default is NO."
   "6) Get-WiFiCoreBroadcasting - Selects first Baselined profile within broadcasting range"

#Get the option
   $nums = @(1,2,3,4,5,6)
   $funcs = @("Get-WifiProfile","Remove-WifiProfile","Make-WiFiProfilesBaseline","Baseline-WiFiProfiles","Get-SSIDProfileInfo","Get-WiFiCoreBroadcasting")
   $option = read-host -Prompt "1-6 or Function Name from above"
    if ($option -in $stoppers) { $input = $option
       break }
    if ($option -in $nums ) {}
    elseif ($option -in $funcs) {}
    else { "invalid" }
#Get any Params
   if (($option -eq 1) -or ($option -eq "Get-WifiProfile")) {
       $ProfName = read-host -Prompt "Enter Profile Name"
   }
   if (($option -eq 2) -or ($option -eq "Remove-WifiProfile")) {
       $ProfName = read-host -Prompt "Enter Profile Name"
   }
   if (($option -eq 4) -or ($option -eq "BaseLine-WiFiProfiles")) {
       $Action = read-host -Prompt "Action YES or NO.  Default NO"
   }
   if (($option -eq 5) -or ($option -eq "Get-SSIDProfileInfo")) {
       $Key = read-host -Prompt "Show Keys YES or NO.  Default NO"
       $ProfName = read-host -Prompt "Profile Name.  Default None"
   }

#Do what is asked
   if (($option -eq 1) -or ($option -eq "Get-WifiProfile")) {
   Get-WifiProfile -Name $ProfName
   }
   if (($option -eq 2) -or ($option -eq "Remove-WifiProfile")) {
   if ($profName.length -gt 0) {
      Remove-WifiProfile -Name $ProfName
      }
      else { "Profile Name Required "}
   }
   if (($option -eq 3) -or ($option -eq "Make-WiFiProfilesBaseline")) {
   Make-WiFiProfilesBaseline
   "Baseline completed"
   }
   if (($option -eq 4) -or ($option -eq "BaseLine-WiFiProfiles")) {
      if ($PathName.length -le 0) {
          BaseLine-WiFiProfiles -Action $Action 
      }
      else {
          BaseLine-WiFiProfiles -Action $Action -BasePath $PathName
      }
   }
   if (($option -eq 5) -or ($option -eq "Get-SSIDProfileInfo")) {
      if ($ProfName.length -le 0) {
         Get-SSIDProfileInfo -Key $key
      }
      else {
         Get-SSIDProfileInfo -Key $key -ProfileName $ProfName
      }
   }
   if (($option -eq 6) -or ($option -eq "Get-WiFiCoreBroadcasting")) {
   Get-WiFiCoreBroadcasting
   }
} While ($input -inotcontains $Stoppers) 