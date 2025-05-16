
@echo off
title 麦克风音量保持脚本
mode con cols=50 lines=18
color 0A


set "NIRCMD_PATH=%windir%\System32\nircmdc.exe"
set "LOOP_INTERVAL=50"  :: 默认间隔50毫秒
set "VOLUME_PERCENT=100"  :: 默认音量百分比
set "VOLUME_VALUE=65535"  :: 默认音量值（65535 = 100%）


fltmc >nul 2>nul || (
    echo 错误: 请右键以管理员身份运行!
    pause
    exit /b 1
)


if not exist "%NIRCMD_PATH%" (
    echo 错误: 未找到 nircmdc.exe!
    echo 请从 http://www.nirsoft.net/utils/nircmd.html 下载并放入System32目录
    pause
    exit /b 1
)


:Configure
cls
echo      
echo ==================================
echo      使用此脚本代表你认识Gardenia
echo       本脚本由Gardenia制作
echo      如果你是付费得到的请找卖方退款
echo    注意：使用此脚本出现的任何问题制作者不负责
echo    使用此脚本不可删除系统temp目录
echo    Gardenia wechat：Gardenia2011422
echo ==================================
echo         麦克风音量保持脚本设置
echo         建议循环间隔设置大于10毫秒（CPU占用小）
echo ==================================
echo 当前设置:
echo   音量: %VOLUME_PERCENT%%%
echo   间隔: %LOOP_INTERVAL% 毫秒
echo ----------------------------------
echo [1] 设置音量百分比 (0-100)
echo [2] 设置循环间隔 (毫秒)
echo [3] 开始运行
echo [4] 退出
echo ==================================

choice /c 1234 /n /m "请选择操作 [1-4]: "

if errorlevel 4 exit /b 0
if errorlevel 3 goto StartLoop
if errorlevel 2 goto SetInterval
if errorlevel 1 goto SetVolume

:SetVolume
cls
echo ==================================
echo          设置音量百分比
echo ==================================
set /p "VOLUME_PERCENT=请输入音量百分比 (0-100): "

set "isNumber=1"
for /f "delims=0123456789" %%i in ("%VOLUME_PERCENT%") do set "isNumber=0"

if %isNumber% equ 0 (
    echo 错误: 请输入纯数字!
    pause
    goto SetVolume
)

if %VOLUME_PERCENT% LSS 0 (
    echo 错误: 音量不能小于0%!
    pause
    goto SetVolume
)

if %VOLUME_PERCENT% GTR 100 (
    echo 错误: 音量不能超过100%!
    pause
    goto SetVolume
)

:: 转换为系统音量值 (0-65535)
set /a VOLUME_VALUE=%VOLUME_PERCENT%*65535/100
echo 已成功设置音量为 %VOLUME_PERCENT%%% (%VOLUME_VALUE%)
pause
goto Configure

:SetInterval
cls
echo ==================================
echo          设置循环间隔
echo ==================================
set /p "LOOP_INTERVAL=请输入循环间隔 (毫秒): "


set "isNumber=1"
for /f "delims=0123456789" %%i in ("%LOOP_INTERVAL%") do set "isNumber=0"

if %isNumber% equ 0 (
    echo 错误: 请输入纯数字!
    pause
    goto SetInterval
)


if %LOOP_INTERVAL% LSS 1 (
    echo 错误: 间隔不能小于1毫秒!
    pause
    goto SetInterval
)

echo 已设置循环间隔为 %LOOP_INTERVAL% 毫秒
pause
goto Configure


:StartLoop
cls
echo ==================================
echo         麦克风音量保持脚本 - 正在运行
echo ==================================
echo 当前时间: %time%
echo 音量设置: %VOLUME_PERCENT%%% (%VOLUME_VALUE%)
echo 循环间隔: %LOOP_INTERVAL% 毫秒
echo ----------------------------------
echo 按 Ctrl+C 关闭所有脚本以及此窗口
echo ==================================


set "loopRunning=1"
echo %loopRunning% > "%temp%\volume_loop_flag.tmp"

call :VolumeLoop


del /f /q "%temp%\volume_loop_flag.tmp" 2>nul
echo.
echo 已返回配置菜单...
pause
goto Configure


:VolumeLoop
if not exist "%temp%\volume_loop_flag.tmp" exit /b 0


"%NIRCMD_PATH%" setsysvolume %VOLUME_VALUE% default_record


"%NIRCMD_PATH%" wait %LOOP_INTERVAL%

goto VolumeLoop
