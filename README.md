## JuanFi onLogin/onLogout v7.1a
- no need to define hotspot folder! (copyright)
- not recommended for beginners!
- for advance users only!
- it uses a powerful system scripting functions!
- totally different kind of login/logout scripts!

### What's in v7 (2023-12-29)
- not compatible with JuanFi Manager APK! { important }
- would cause conflict if using the other brownout script! { important }
- uses user-email to check valid entry! { NEW }
- cancel user-login if invalid user email/profile! { NEW }
- cancel user-login if invalid user comment! { NEW }
- dual data-file created (mac/user) if mac!=user! { NEW }
- system scheduler on-event minimize/functionalize! { NEW }
- user scheduler is created first ( full/normal/lite )
- cancel user-login if scheduler not created ( full/normal )
- fix validity if validity<user-time ( full/normal/lite )
- extend code is used for log purposes ( full/normal/lite )
- create logs for AddNew/Extend user ( full/normal )
- create logs for repor ( full/normal )
- create logs for error ( full )
- user comment details are added ( full/normal )
- auto create data folder if missing ( full )
- auto create sales files if missing ( full )
- sales update functionalize ( full/normal )
- telegram reporting ( full/normal )

### Transistion/Migration:
- the new onLogin/onLogout v7 script will take effect on new users.
- old active users will still use the old scheduler script.
- just leave those old active users as-is until they expire!

### Tested On:
- hAP Lite 6.48.7 Stable
- hEX GR3 7.12.1 Stable
- RB5009 7.8 Long Term
- CCR1009 6.49.10 Long Term

### WARNING:
- test first before deploy!

### Author:
- Chloe Renae & Edmar Lozada

### Facebook Contact:
- https://www.facebook.com/chloe.renae.2000

### RealTalk:
- kung naka tulong kami at gamit ninyo script namin,
  magpa gcash naman kayo sa amin kahit kunti lang!

### v7.2a:
- v7.2a is more advance/detailed, only for my gcash customers.

### Follow these steps:

#### Step 1: Copy script below and paste to mikrotik terminal. ( juanfi_hs_full_v71_script.txt )
note: Needed JuanFi Functions use by onLogin/onLogout
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
/ip hotspot active remove [find user=\$iUser]
local iUsrTime 0s; local iUseTime 0s; local aUser
local iCExpire \"Validity\"
do {
if ([/ip hotspot user find name=\$iUser]!=\"\") do={
  set aUser [/ip hotspot user get \$iUser]
  set iUsrTime (\$aUser->\"limit-uptime\")
  set iUseTime (\$aUser->\"uptime\")
}
if (\$iUsrTime<=\$iUseTime) do={ set iCExpire \"TimeLimit\" }
log debug \"EXPIRE USER ( \$iCExpire ) => user=[\$iUser] mac=[\$iDMac] usertime=[\$iUsrTime] uptime=[\$iUseTime]\"
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

# === eGetDateTime ===
{ local sName "eGetDateTime"
  local eSource "# $sName #\r
# by: Chloe Renae & Edmar Lozada\r
# ------------------------------\r
local i2 \$2
local dD [/system clock get date]
local nY; local nD; local nM; local cM; local iRet \"\"
local sM \"...JanFebMarAprMayJunJulAugSepOctNovDec\"
if ([len \$dD]=10) do={
  set nY [pick \$dD 0  4]; set nD [pick \$dD 8 10]; set nM [pick \$dD 5  7]
}
if ([len \$dD]=11) do={
  set nY [pick \$dD 7 11]; set nD [pick \$dD 4  6]; set cM [pick \$dD 0  3]
  set nM [pick (100+([find \$sM [pick \$dD 1 3]]/3)) 1 3]
}
local nLen [len \$i2]
if (\$nLen=14) do={set nM [pick \$i2 0  2]; set nD [pick \$i2 3  5]}
if (\$nLen=15 or \$nLen=20) do={
  set nM [pick (100+([find \$sM [pick \$dD 1 3]]/3)) 1 3]
  set nD [pick \$i2 4  6]
}
if (\$nLen=19) do={set nY [pick \$i2 0  4]; set nM [pick \$i2 5  7]; set nD [pick \$i2 8 10]}
if (\$nLen=20) do={set nY [pick \$i2 7 11]}
if (\$1=\"D\") do={set iRet \"\$nY-\$nM-\$nD\"}
if (\$1=\"T\") do={set iRet [pick \$i2 (\$nLen-8) \$nLen]}
return \$iRet\r
# ------------------------------\r\n"
if ([/system script find name="ss-$sName"]="") do={/system script add name="ss-$sName"}
/system script set [find name="ss-$sName"] source=$eSource owner="system function" comment="( function_system-12: $sName )"
}

```

#### Step 2: Copy script below and paste to hotspot user profile onLogin. ( juanfi_hs_full_71a_onLogin.txt )
```bash
# juanfi_hs_onLogin_71a_full
# by: Chloe Renae & Edmar Lozada
# ------------------------------

local iUser $username
local iDMac $"mac-address"
local iDInt $interface
local aUser [/ip hotspot user get $iUser]
local iMail ($aUser->"email")
local eLogoutUser do={
  /ip hotspot active remove [find user=$iUser]
  /ip hotspot cookie remove [find user=$iUser]
  /ip hotspot cookie remove [find mac-address=$iDMac]
}
# Cancel user-login if Invalid User eMail/Profile
if (!($iMail~"new" || $iMail~"extend" || $iMail~"active")) do={
  log error "( $iUser ) ONLOGIN ERROR: [$iMail] => INVALID EMAIL!"
  /ip hotspot user set [find name=$iUser] disable=yes
  $eLogoutUser iUser=$iUser iDMac=$iDMac; return 0
}

# Check Valid Entry
if ($iMail~"new" || $iMail~"extend") do={
  local aNote [toarray ($aUser->"comment")]
  local iValidity [totime ($aNote->0)]
  local iSalesAmt [tonum ($aNote->1)]
  local iExtUCode ($aNote->2)
  local iVendoNme ($aNote->3)
  # Cancel user-login if Invalid Comment
  if (!($iValidity>=0 && $iSalesAmt>=0 && ($iExtUCode=0 or $iExtUCode=1))) do={
    log error "( $iUser ) ONLOGIN ERROR! val:[$iValidity] amt:[$iSalesAmt] ext:[$iExtUCode] => INVALID COMMENT!"
    /ip hotspot user set [find name=$iUser] disable=yes
    $eLogoutUser iUser=$iUser iDMac=$iDMac; return 0
  }
  local eReplace do={
    local iRet
    for i from=0 to=([len $1]-1) do={
      local x [pick $1 $i]
      if ($x=$2) do={set x $3}
      set iRet ($iRet.$x)
    }; return $iRet
  }
  local iRandMac ("26AE"~[pick $iDMac 1 2])
  local iMacFile [$eReplace $iDMac ":" ""]
  local iUsrTime ($aUser->"limit-uptime")
  local iHotSpot [/ip hotspot profile get [.. get [find interface=$iDInt] profile] html-directory]

  # Add User Scheduler
  do {
  if ([/system scheduler find name=$iUser]="") do={
    /system scheduler add name=$iUser interval=0 \
    on-event=("# EXPIRE ( $iUser ) #\r\n".\
              "local iUser \"$iUser\"; local iDMac \"$iDMac\"\r\n".\
              "local iMacFile \"$iMacFile\"; local iHotSpot \"$iHotSpot\"\r\n".\
              "do {\r\n".\
              "local eUserExpire [parse [/system script get ss-1Fi-eUserExpire source]]\r\n".\
              "\$eUserExpire \$iUser \$iDMac \$iMacFile \$iHotSpot\r\n".\
              "} on-error={log error \"( \$iUser ) SYSHED ERROR! Expire User Module\"}\r\n".\
              "# END #\r\n")
    local x 10;while (($x>0)&&([/system scheduler find name=$iUser]="")) do={set x ($x-1);delay 1s}
    set iExtUCode 0
  }
  # Cancel user-login if user-scheduler NOT FOUND!
  if ([/system scheduler find name=$iUser]="") do={
    log error "( $iUser ) ONLOGIN ERROR! /system scheduler [$iUser] => NOT FOUND!"
    $eLogoutUser iUser=$iUser iDMac=$iDMac; return 0
  }
  } on-error={log error "( $iUser ) ONLOGIN ERROR! Add User Scheduler Module"}

  # Update User Validity/Interval/Comment/eMail
  if ($iExtUCode=0) do={ set iExtUCode "ADDNEW USER" }
  if ($iExtUCode=1) do={ set iExtUCode "EXTEND USER" }
  log warning "$iExtUCode ( $interface ) => user=[$iUser] mac=[$iDMac] usertime=[$iUsrTime] amt=[$iSalesAmt]"
  set iValidity ($iValidity + [/system scheduler get [find name=$iUser] interval])
  # if ($iValidity = 0s and $iUsrTime > 1d) do={ set iValidity $iUsrTime }; # BUG FIX (temporary)
  if ($iValidity != 0s and $iValidity < $iUsrTime) do={ set iValidity ($iUsrTime + $iValidity) }; # BUG FIX
  /system scheduler set [find name=$iUser] interval=$iValidity
  /ip hotspot user set [find name=$iUser] comment=""
  /ip hotspot user set [find name=$iUser] email="$iSalesAmt@juanfi.$iMacFile.active"

  # Set User Date/Time Variables
  local eGetDate [parse [/system script get ss-eGetDateTime source]]
  local iDateBeg [/system scheduler get [find name=$iUser] start-date]
  local iTimeBeg [/system scheduler get [find name=$iUser] start-time]
  local iUsrBeg0 ([$eGetDate "D" $iDateBeg]." ".[pick $iTimeBeg 0 5])
  local iNextRun [/system scheduler get [find name=$iUser] next-run]
  local iDateEnd [$eGetDate "D" $iNextRun]
  local iTimeEnd [$eGetDate "T" $iNextRun]
  local iUsrEnd0 "NO EXPIRATION"
  if ([len $iNextRun]>1) do={
    set iUsrEnd0 ($iDateEnd." ".[pick $iTimeEnd 0 5])
  }
  log info "( $iUser ) beg=[$iUsrBeg0] end=[$iUsrEnd0] validity=[$iValidity] vendo=[$iVendoNme]"

  # Add User Data File
  local eAddDatFiles [parse [/system script get ss-1Fi-eAddDatFiles source]]
  $eAddDatFiles $iUser $iMacFile $iUsrEnd0 $iHotSpot
  if ($iUser!=$iMacFile) do={ $eAddDatFiles $iUser $iUser $iUsrEnd0 $iHotSpot }

  # Update Sales ( Today/Month )
  local eAddSales [parse [/system script get ss-1Fi-eAddSales source]]
  local iSalesToday [$eAddSales $iUser $iSalesAmt "todayincome" "JuanFi Sales Daily ( TOTAL )"]
  local iSalesMonth [$eAddSales $iUser $iSalesAmt "monthlyincome" "JuanFi Sales Monthly ( TOTAL )"]

  # Telegram Reporting
  do {
  local isTelegram 0 ;# 0=disable or 1=enable
  if ($isTelegram=1) do={
    local iTGBotToken "xxxxxxxxxx:xxxxxxxxxxxxx-xxxxxxxxxxxxxxx-xxxxx" ;# Telegram Bot Token
    local iTGrpChatID "xxxxxxxxxxxxxx" ;# Telegram Group Chat ID
    local iUActive [/ip hotspot active print count-only]
    local iDevName [pick [/ip dhcp-server lease get [find mac-address=$iDMac] host-name] 0 15]
    local iMessage ("<<== $iExtUCode ==>>%0A".\
                    "User: $iUser%0A".\
                    "IP: $address%0A".\
                    "MAC: $iDMac%0A".\
                    "Device: $iDevName%0A".\
                    "Active Users: $iUActive%0A%0A".\
                    "Vendo Name : $iVendoNme%0A".\
                    "User Time  : $iUsrTime%0A".\
                    "Beg: $iUsrBeg0%0A".\
                    "End: $iUsrEnd0%0A".\
                    "Sale Amount: $iSalesAmt%0A".\
                    "Total Today: $iSalesToday%0A".\
                    "Total Month: $iSalesMonth%0A".\
                    "Vendo Today: $iVendoToday%0A".\
                    "Vendo Month: $iVendoMonth%0A".\
                    "<<=====================>>")
    local iMessage [$eReplace ($iMessage) " " "%20"]
    do {/tool fetch url="https://api.telegram.org/bot$iTGBotToken/sendmessage?chat_id=$iTGrpChatID&text=$iMessage" keep-result=no} on-error={}
  }
  } on-error={log error "( $iUser ) ONLOGIN ERROR! Telegram Reporting Module"}

  # User Comment Info
  /ip hotspot user  set [find name=$iUser] comment="+ end=[$iUsrEnd0] beg=[$iUsrBeg0] mac=[$iDMac] validity=[$iValidity] vlan=[ $interface ] amt=[$iSalesAmt]"
  /system scheduler set [find name=$iUser] comment="+ ( $interface ) beg=[$iUsrBeg0] end=[$iUsrEnd0] mac=[$iDMac] usertime=[$iUsrTime] amt=[$iSalesAmt]"
}

```

#### Step 3: Copy script below and paste to hotspot user profile onLogout. ( juanfi_hs_full_71a_onLogout.txt )
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
