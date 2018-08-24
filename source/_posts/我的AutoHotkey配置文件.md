---
title: 我的AutoHotkey配置文件
categories: 编程
copyright: true
---

## AutoHotkey安装与配置
## Quick Start
###1. AutoHotkey安装

官方文档 (https://www.autohotkey.com/docs/Tutorial.htm)
官方下载地址 (https://www.autohotkey.com/) 

###2. 操作说明

 - 可以使用/alt 打印alt键相关快捷键
 - 可以使用/win 打印windows快捷键
 - 可以使用/other 打印其他快捷键 
 - 注意在脚本中回车用“`n”表示，加号用“{+}”表示
 - 将上述脚本保存为my.ahk文件保存到你的工作目录，然后双击运行。
 - 将my.ahk文件创建快捷方式，放到windows启动文件夹中，即可实现开机后自动运行AutoHotkey

<!-- more -->

###3. 我的配置文件

``` bash
;Notes: #==win !==Alt,  ^==Ctr, +==shift

;===========================================
#t::Run https://www.travis-ci.org/
#b::Run https://www.baidu.com/
#h::Run https://zhanglingsi.github.io/
#c::Run http://zhanglingsi.coding.me/zhanglingsi/
#j::Run https://github.com/zhanglingsi/
; 工作相关
#v::Run http://183.230.93.167:18082/dacp/login
#n::Run http://112.25.233.117:58080/OnlineServer/FrameAction_myResources.action
#i::Run https://tinypng.com/
#o::Run http://oa.asiainfo.com/
#s::winset,AlwaysOnTop,, A
#q::Send !{F4}
;======================启动应用程序=====================
!n::run notepad
!b::run, D:\Program Files (x86)\cmd_markdown_win64\Cmd Markdown.exe
!r::run, D:\Program Files\cmder\Cmder.exe
!w::run, C:\Program Files (x86)\Tencent\WeChat\WeChat.exe
!s::run, D:\Program Files\UltraEdit-32\Uedit32.exe
!t::run, D:\Program Files\SecureCRT_x64\SecureCRTPortable.exe
!i::run, D:\Program Files\JetBrains\IntelliJ IDEA 2017.1.4\bin\idea64.exe
!m::run, C:\Program Files\SQLyog\SQLyog.exe
!o::run, C:\Program Files (x86)\PLSQL Developer\plsqldev.exe
!f::run, D:\Program Files\Foxmail 7.2\Foxmail.exe
!e::run, C:\Program Files\internet explorer\iexplore.exe
!d::run, D:\Program Files\Microsoft VS Code\Code.exe
!q::run, C:\Program Files (x86)\Tencent\TIM\Bin\TIM.exe
!x::WinMinimize A
!z::WinMaximize A
!c::WinRestore A
!1::send #{Tab} 
!3::send #^{right} 
!2::send #^{left} 
;===================拷贝文件路径============================
^+c::
; null= 
send ^c
sleep,200
clipboard=%clipboard% ;%null%
tooltip,%clipboard%
sleep,500
tooltip,
return
;================================================
;replace CapsLock to LeftEnter; CapsLock = Alt CapsLock
$CapsLock::Enter
LAlt & Capslock::SetCapsLockState, % GetKeyState("CapsLock", "T") ? "Off" : "On"
!u::Send ^c !{tab} ^v
;================================================
::/cmail::zhanglingsi@asiainfo.com
::/mail::zhanglingsi@aliyun.com
::/sf::select * from  t ;
::/sfw::select * from  t where 1=1 and ;
::/scf::select count(*) from  t ;
::/scfg::select count(*) from  t where 1=1 group by t.;
::/js::javascript:;
::/win:: Win {+} e：打开资源管理器;`nWin` {+} d：显示桌面;`nWin` {+} f：打开查找对话框;`nWin {+} r：打开运行对话框;`nWin {+} l：锁定电脑;`nWin {+} u: 打开控制面板－轻松使用设置中心;`n`nwin {+} t：打开网页travis-ci;`nwin {+} b：打开网页baidu;`nwin {+} h：打开网页blog for github;`nwin {+} c：打开网页blog for coding;`nwin {+} j：打开网页my github;`nwin {+} v：打开网页DACP;`nwin {+} i：打开网页tinypng;`nwin {+} o：打开网页OA;`nwin {+} s：置顶当前窗口;`nwin {+} q：关闭应用程序;`nwin {+} z：激活chrome窗口;`n
::/alt:: ALT {+} n： 打开notepad;`nALT {+} b： 打开打开Markdown;`nALT {+} r： 打开Cmder;`nALT {+} w： 打开微信;`nALT {+} s： 打开Uedit32;`nALT {+} t： 打开CRT;`nALT {+} i： 打开idea;`nALT {+} m： 打开SQLyog;`nALT {+} o： 打开plsqldev;`nALT {+} f： 打开Foxmail;`nALT {+} e： 打开iexplore;`nALT {+} d： 打开VSCode;`nALT {+} q： 打开TIM;`nALT {+} x： 最小化当前窗口;`nALT {+} z： 最大化当前窗口;`nALT {+} c： 还原当前窗口;`nALT {+} 1： 打开虚拟桌面;`nALT {+} 3： 向右切换虚拟桌面;`nALT {+} 2： 向左切换虚拟桌面;`n
::/other::  Ctrl {+} shift {+} C：拷贝文件路径;`nCtrl {+}  Win {+}  C：拷贝取色;`n
;=====================拷贝鼠标取色==============================
^#c::
MouseGetPos, mouseX, mouseY
PixelGetColor, color, %mouseX%, %mouseY%, RGB
StringRight color,color,6
clipboard = %color%
return
;====================切换chrome===============================
#z::
IfWinNotExist ahk_class Chrome_WidgetWin_1
{
    Run "C:\Users\Administrator\AppData\Local\Google\Chrome\Application\chrome.exe"
    WinActivate
}
Else IfWinNotActive ahk_class Chrome_WidgetWin_1
{
    WinActivate
}
Else
{
    WinMinimize
}
Return
;===================================================
;左侧shift+鼠标滚轮调整窗口透明度（设置30-255的透明度，低于30基本上就看不见了，如需要可自行修改)
;
;使用说明：
;   ◆左侧shift+滚轮下滑：减少透明度，一次10
;   ◆左侧shift+滚轮上滑：增加透明度，一次20
;   ◆左侧shift+中键按下：恢复透明度至255(完全不透明).
;
~LShift & WheelUp::
; 透明度调整，增加。
WinGet, Transparent, Transparent,A
If (Transparent="")
    Transparent=255
    ;Transparent_New:=Transparent+10
    Transparent_New:=Transparent+20    ;◆透明度增加速度。
    If (Transparent_New > 254)
                    Transparent_New =255
    WinSet,Transparent,%Transparent_New%,A
 
    tooltip now: ▲%Transparent_New%`nmae: __%Transparent%  ;查看当前透明度（操作之后的）。
    ;sleep 1500
    SetTimer, RemoveToolTip_transparent_Lwin__2016.09.20, 1500  ;设置统一的这个格式，label在最后。
return
 
~LShift & WheelDown::
;透明度调整，减少。
WinGet, Transparent, Transparent,A
If (Transparent="")
    Transparent=255
    Transparent_New:=Transparent-10  ;◆透明度减少速度。
    ;msgbox,Transparent_New=%Transparent_New%
            If (Transparent_New < 30)    ;◆最小透明度限制。
                    Transparent_New = 30
    WinSet,Transparent,%Transparent_New%,A
    tooltip now: ▲%Transparent_New%`nmae: __%Transparent%  ;查看当前透明度（操作之后的）。
    ;sleep 1500
    SetTimer, RemoveToolTip_transparent_Lwin__2016.09.20, 1500  ;设置统一的这个格式，label在最后。
return
 
;设置Lwin &Mbutton直接恢复透明度到255。
~Lshift & Mbutton:: 
WinGet, Transparent, Transparent,A
WinSet,Transparent,255,A 
tooltip ▲Restored ;查看当前透明度（操作之后的）。
;sleep 1500
SetTimer, RemoveToolTip_transparent_Lwin__2016.09.20, 1500  ;设置统一的这个格式，label在最后。
return
 
 
removetooltip_transparent_Lwin__2016.09.20:     ;LABEL
tooltip
SetTimer, RemoveToolTip_transparent_Lwin__2016.09.20, Off
return
;===================================================
; 按住空格不放，搭配edsf实现上下左右方向键，搭配cv实现backspace和del功能，搭配wr，实现home和end功能：

#NoEnv 
; #Warn
SendMode Input
SetWorkingDir %A_ScriptDir% 

AnyKeyPressedOtherThanSpace(mode = "P") {
    keys = 1234567890-=qwertyuiop[]\asdfghjkl;'zxcvbnm,./
    Loop, Parse, keys
    {        
        isDown :=  GetKeyState(A_LoopField, mode)
        if(isDown)
            return True
    }   
    return False
}

Space Up::
    space_up := true
    Send, {F18}
    return
Space::
    if AnyKeyPressedOtherThanSpace(){
        SendInput, {Blind}{Space}
        Return
    }
    space_up := False
    inputed := False
    input, UserInput, L1 T0.1, {F18}
    if (space_up) {
        Send, {Blind}{Space}
        return
    }else if (StrLen(UserInput) == 1){
        Send, {Space}%UserInput%
        return
    }
    while true{
        input, UserInput, L1, {F18}
        if (space_up) {
            if (!inputed){
                Send, {Blind}{Space}
            }
            break
        }else if (StrLen(UserInput) == 1) {
            inputed := True
            StringLower, UserInput, UserInput
            if (UserInput == "e")
                Send, {Blind}{Up}
            else if (UserInput == "d")
                Send, {Blind}{Down}
            else if (UserInput == "s")
                Send, {Blind}{Left}
            else if (UserInput == "f")
                Send, {Blind}{Right}
            else if (UserInput == "w")
                Send, {Blind}{Home}
            else if (UserInput == "r")
                Send, {Blind}{End}
            else if (UserInput == "c")
                Send, {Blind}{BS}
            else if (UserInput == "v")
                Send, {Blind}{DEL}
            else
                Send, {Blind}%UserInput%
        }
    }
    return
;===================================================
~lbutton & enter::  
exitapp  
~WheelUp::  
if (existclass("ahk_class Shell_TrayWnd")=1)  
Send,{Volume_Up}  
Return  
~WheelDown::  
if (existclass("ahk_class Shell_TrayWnd")=1)  
Send,{Volume_Down}  
Return  
~MButton::  
if (existclass("ahk_class Shell_TrayWnd")=1)  
Send,{Volume_Mute}  
Return  

Existclass(class)  
{  
MouseGetPos,,,win  
WinGet,winid,id,%class%  
if win = %winid%  
Return,1  
Else  
Return,0  
}
;===================================================

```

马上丢掉你是鼠标，快来体验飞一般的感觉吧！你懂的！


 
