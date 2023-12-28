# JuanFi onLogin/onLogout v7
- no need to define hotspot folder! (copyright)
- only for advance users!
- not recommended for beginners!
- it uses a powerful system scripting functions!
- totally different kind of login/logout scripts!
### What's in v7 (2023-12-29)
- not compatible with JuanFi Manager APK { important }
- uses user-email to check valid entry! { NEW }
- cancel user=login if invalid user email/profile! { NEW }
- cancel user-login if invalid user comment! { NEW }
- dual data-file created (mac/user) if mac!=user! { NEW }
- system scheduler on-event minimize/functionalize! { NEW }
- fix error on "0" validity ( full/normal/lite )
- user comment details added ( full/normal )
- create logs for full reporting/monitoring ( full/normal )
- user scheduler is created first ( full/normal/lite )
- cancel user-login if scheduler not created ( full/normal )
- create logs for AddNew/Extend user ( full/normal )
- extend code is used ( full/normal/lite )
- auto create data folder if missing ( full )
- auto create sales files if missing ( full )
- sales update functionalize ( full/normal )
- telegram reporting ( full/normal )
### Tested On:
- hEX GR3 7.12.1 Stable
- RB5009 7.8 Long Term
- CCR1009 6.49.10 Long Term
### WARNING:
- test first before deploy!
### Author:
- Chloe Renae & Edmar Lozada
### Facebook Contact:
- https://www.facebook.com/chloe.renae.2000

#### Step 1: Copy script below and paste to mikrotik terminal. ( JuanFi Scripts )
note: Needed JuanFi Functions for onLogin/onLogout
```bash
# ==============================
# Miktrotik JuanFi Scripts
# by: Chloe Renae & Edmar Lozada
# ==============================

{ local sName "1Fi-eUserExpire"
  local sSource "# $sName #\r
# by: Chloe Renae & Edmar Lozada\r
# ------------------------------\r
local iUser \$1; local iDMac \$2; local iMacFile \$3; local iHotSpot \$4
local iUsrTime 0s; local iUseTime 0s; local aUser
local iCExpire \"Validity\"
/ip hotspot active remove [find user=\$iUser]
do {
if ([/ip hotspot user find name=\$iUser]!=\"\") do={
  set aUser [/ip hotspot user get \$iUser]
  set iUsrTime (\$aUser->\"limit-uptime\")
  set iUseTime (\$aUser->\"uptime\")
  local eTime2DT [parse [/system script get ss-eTime2DT source]]
  set iUsrTime [\$eTime2DT \$iUsrTime]; set iUseTime [\$eTime2DT \$iUseTime]
}
if (\$iUsrTime<=\$iUseTime) do={ set iCExpire \"TimeLimit\" }
log warning \"EXPIRE USER ( \$iCExpire ) => user=[\$iUser] mac=[\$iDMac] usertime=[\$iUsrTime] uptime=[\$iUseTime]\"
/ip hotspot cookie remove [find user=\$iUser]
/ip hotspot cookie remove [find mac-address=\$iDMac]
/system scheduler  remove [find name=\$iUser]
/ip hotspot user   remove [find name=\$iUser]
/file remove [find name=\"\$iHotSpot/data/\$iUser.txt\"]
/file remove [find name=\"\$iHotSpot/data/\$iMacFile.txt\"]
} on-error={log error \"( \$iUser ) SYSCRIPT ERROR! Expire User Module\"}
# ------------------------------\r\n"
if ([/system script find name="ss-$sName"]="") do={/system script add name="ss-$sName"}
/system script set [find name="ss-$sName"] source=$sSource owner="juanfi function" comment="( function_juanfi-01: $sName )"
}

{ local sName "1Fi-eAddDatFiles"
  local sSource "# $sName #\r
# by: Chloe Renae & Edmar Lozada\r
# ------------------------------\r
local iUser \$1; local iDatFile \$2; local iUsrEnd \$3; local iHotSpot \$4
if ([/file find name=\"\$iHotSpot\"]!=\"\") do={
if ([/file find name=\"\$iHotSpot/data\"]=\"\") do={
  log error \"( \$iUser ) ONLOGIN: /file [\$iHotSpot/data/] => AUTOCREATE!\"
  do { /tool fetch dst-path=(\"\$iHotSpot/data/.\") url=\"https://127.0.0.1/\" } on-error={ }
  local x 10;while ((\$x>0)&&([/file find name=\"\$iHotSpot/data\"]=\"\")) do={set x (\$x-1);delay 1s}
}
} else={log error \"( \$iUser ) ONLOGIN ERROR! /file [\$iHotSpot] => NOT FOUND!\"}
do {
if ([/file find name=\"\$iHotSpot\"]!=\"\") do={
if ([/file find name=\"\$iHotSpot/data\"]!=\"\") do={
/file print file=\"\$iHotSpot/data/\$iDatFile.txt\" where name=\"\$iDatFile.txt\"
local x 10;while ((\$x>0)&&([/file find name=\"\$iHotSpot/data/\$iDatFile.txt\"]=\"\")) do={set x (\$x-1);delay 1s}
if ([/file find name=\"\$iHotSpot/data/\$iDatFile.txt\"]!=\"\") do={
/file set \"\$iHotSpot/data/\$iDatFile\" contents=\"\$iUser#\$iUsrEnd\"
} else={log error \"( \$iUser ) ONLOGIN ERROR! /file [\$iHotSpot/data/\$iDatFile.txt] => NOT FOUND!\"}
} else={log error \"( \$iUser ) ONLOGIN ERROR! /file [\$iHotSpot/data] => NOT FOUND!\"}
} else={log error \"( \$iUser ) ONLOGIN ERROR! /file [\$iHotSpot] => NOT FOUND!\"}
} on-error={log error \"( \$iUser ) ONLOGIN ERROR! AutoCreate User Data File Module\"}
# ------------------------------\r\n"
if ([/system script find name="ss-$sName"]="") do={/system script add name="ss-$sName"}
/system script set [find name="ss-$sName"] source=$sSource owner="juanfi function" comment="( function_juanfi-02: $sName )"
}

{ local sName "1Fi-eAddSales"
  local sSource "# $sName #\r
# by: Chloe Renae & Edmar Lozada\r
# ------------------------------\r
local iUser \$1; local iSalesAmt \$2; local iSalesName \$3; local iSalesComment \$4
do {
if ([/system script find name=\$iSalesName]=\"\") do={
  log error \"( \$iUser ) ONLOGIN ERROR! /system script [\$iSalesName] => AUTOCREATE!\"
  /system script add name=\$iSalesName source=\"0\"
  local x 10;while ((\$x>0)&&([/system script find name=\$iSalesName]=\"\")) do={set x (\$x-1);delay 1s}
}
} on-error={log error \"( \$iUser ) ONLOGIN ERROR! AutoCreate \$iSalesName Module\"}
local iTotalAmt
do {
if ([/system script find name=\$iSalesName]!=\"\") do={
  local iSaveAmt [tonum [/system script get [find name=\$iSalesName] source]]
  set \$iTotalAmt ( \$iSalesAmt + \$iSaveAmt )
  /system script set [find name=\$iSalesName] source=\"\$iTotalAmt\" comment=\$iSalesComment
} else={log error \"( \$iUser ) ONLOGIN ERROR! /system script [\$iSalesName] => NOT FOUND!\"}
} on-error={log error \"( \$iUser ) ONLOGIN ERROR! Update \$iSalesName Module\"}
return \$iTotalAmt
# ------------------------------\r\n"
if ([/system script find name="ss-$sName"]="") do={/system script add name="ss-$sName"}
/system script set [find name="ss-$sName"] source=$sSource owner="juanfi function" comment="( function_juanfi-03: $sName )"
}

```

#### Step 2: Copy script below and paste to hotspot user profile onLogin. ( onLogin Script )
```bash
comming soon!
```

#### Step 3: Copy script below and paste to hotspot user profile onLogout. ( onLogout Script )
```bash
# juanfi_hs_onLogout_71a_full
# by: Chloe Renae & Edmar Lozada
# ------------------------------

# Expire User
local iUser $username
local aUser [/ip hotspot user get $iUser]
local iUsrTime ($aUser->"limit-uptime")
local iUseTime ($aUser->"uptime")

# Check Expiration
do {
if ($cause="traffic limit reached" || ($iUsrTime>0 && $iUsrTime<=$iUseTime)) do={
  local iDMac $"mac-address"
  local iDInt $interface
  local iHotSpot [/ip hotspot profile get [.. get [find interface=$iDInt] profile] html-directory]
  local iMacFile ""
  for i from=0 to=([len $iDMac]-1) do={
    local x [pick $iDMac $i]
    if ($x=":") do={set x ""}
    set iMacFile ($iMacFile.$x)
  }
  local eUserExpire [parse [/system script get ss-1Fi-eUserExpire source]]
  $eUserExpire $iUser $iDMac $iMacFile $iHotSpot
}
} on-error={log error "( $iUser ) ONLOGOUT ERROR! Expire User Module"}

```
