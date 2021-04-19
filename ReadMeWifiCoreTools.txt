Wifi Profile Management
.Synopsis
   This function group provides a set of tools to assist with the maintenance of a healthy WiFi profile list.
   It also assist with the management and core activities associated with WiFi profiles.
.Description
   Get-WifiProfile - gets the list of Wifi Profiles currently stored on this computer.
   Remove-WifiProfile - removes a named profile from the local machine.
   Make-WiFiProfilesBaseline - Takes the current profiles and treats that as the core set for future baselining 
         of for selecting preferences to connect with.
   Baseline-WiFiProfiles - Indicates profiles that in excess of the Baslined core profiles and allows you to 
         choose to remove profiles you may have added over time.
   Get-SSIDProfileInfo - Provides detailed information of the characteristics of current WIFI profiles
   Get-WiFiCoreBroadcasting - gets the first Core wifi profile that is currently within broadcasting range  

Presumes running in PS ISE or PowerShell Command window.

Hash Check Values for this Tool and its files available at http://web.ncf.ca/bv178/HashChecks.html