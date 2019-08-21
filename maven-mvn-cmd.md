## windows mvn.cmd文件分析
 * 作用mvn调用命令
 * 位于
   + %MAVEN_HOME%/bin/mvn.cmd
 
### mvn 启动过程
 * 整理思路
   1. 判断执行前置脚本
     + USERPROFILE目录。即操作系统安装目录下的当前用户下的目录
   2. 获取java命令
   3. 获取maven_home
   4. 查找项目目录
     + 如果配置了MAVEN_BASEDIR变量，以这个变量值做为项目目录。查找结束
     + 如果配置了-f 参数。
        - 如果参数无效，则使用当前目录做为项目目录。查找结束
        - 如果参数为文件，则找到文件所在目录为项目目录，查找结束
        - 如果参数为目录，则做为项目目录，查找结束
   5. 确定项目目录
     + 在找到的项目目录中查找.mvn目录
        - 如果没有.mvn目录，则一直向上层目录查找，直到找到，则做为真正项目目录。
     + 在所有项目目录路径中都没有找到.mvn目录，则使用当前命令执行所在目录为真正项目目录。
     
   5. 启动程序
   6. 判断执行后置脚本
     + USERPROFILE目录。即操作系统安装目录下的当前用户下的目录
   7. 判断是否需要暂停
   8. 退出
 * 启动参数
   + 执行命令如：mvn clean package -DskipTests -f other_pom.xml -P staging
   + JAVACMD: 表示java命令
   + JVM_CONFIG_MAVEN_PROPS：.mvn/jvm.config 配置的jvm参数
   + MAVEN_OPTS：自定义配置的jvm参数
   + MAVEN_DEBUG_OPTS：maven代码debug端口配置
   + -classpath %CLASSWORLDS_JAR%： 启动类
   + Dclassworlds.conf=%MAVEN_HOME%\bin\m2.conf 启动类需要的配置文件
   + -Dmaven.home=%MAVEN_HOME%：maven启动需要的参数
   + -Dmaven.multiModuleProjectDirectory=%MAVEN_PROJECTBASEDIR%：maven启动需要的参数
   + CLASSWORLDS_LAUNCHER 启动类
   + MAVEN_CMD_LINE_ARGS：命令行参数如：clean package -DskipTests -f other_pom.xml -P staging
   ``` 
         "%JAVACMD%" ^
             %JVM_CONFIG_MAVEN_PROPS% ^
             %MAVEN_OPTS% ^
             %MAVEN_DEBUG_OPTS% ^
             -classpath %CLASSWORLDS_JAR% ^
             "-Dclassworlds.conf=%MAVEN_HOME%\bin\m2.conf" ^
             "-Dmaven.home=%MAVEN_HOME%" ^
             "-Dlibrary.jansi.path=%MAVEN_HOME%\lib\jansi-native" ^
             "-Dmaven.multiModuleProjectDirectory=%MAVEN_PROJECTBASEDIR%" ^
             %CLASSWORLDS_LAUNCHER% %MAVEN_CMD_LINE_ARGS%
   ```
  
``` 

@REM Begin all REM lines with '@' in case MAVEN_BATCH_ECHO is 'on'
@echo off
@REM enable echoing my setting MAVEN_BATCH_ECHO to 'on'
@if "%MAVEN_BATCH_ECHO%"=="on" echo %MAVEN_BATCH_ECHO%

@REM Execute a user defined script before this one

# 如果配置了这个属性，则跳过自定义配置
if not "%MAVEN_SKIP_RC%"=="" goto skipRcPre
@REM check for pre script, once with legacy .bat ending and once with .cmd ending

# 自定义配置
if exist "%USERPROFILE%\mavenrc_pre.bat" call "%USERPROFILE%\mavenrc_pre.bat"
if exist "%USERPROFILE%\mavenrc_pre.cmd" call "%USERPROFILE%\mavenrc_pre.cmd"
:skipRcPre

@setlocal

set ERROR_CODE=0

@REM ==== START VALIDATION ====
if not "%JAVA_HOME%"=="" goto OkJHome
for %%i in (java.exe) do set "JAVACMD=%%~$PATH:i"
goto checkJCmd

# 查找java命令
:OkJHome
set "JAVACMD=%JAVA_HOME%\bin\java.exe"

:checkJCmd
if exist "%JAVACMD%" goto chkMHome

echo The JAVA_HOME environment variable is not defined correctly >&2
echo This environment variable is needed to run this program >&2
echo NB: JAVA_HOME should point to a JDK not a JRE >&2
goto error

# 查找maven_home. 即当前脚本mvn.cmd所在目录的上层目录
:chkMHome
set "MAVEN_HOME=%~dp0.."
if not "%MAVEN_HOME%"=="" goto stripMHome
goto error

# 去掉目录末尾 \
:stripMHome
if not "_%MAVEN_HOME:~-1%"=="_\" goto checkMCmd
set "MAVEN_HOME=%MAVEN_HOME:~0,-1%"
goto stripMHome

# 检查mvn.cmd
:checkMCmd
if exist "%MAVEN_HOME%\bin\mvn.cmd" goto init
goto error
@REM ==== END VALIDATION ====

:init

# 获取命令行参数
set MAVEN_CMD_LINE_ARGS=%*

@REM Find the project basedir, i.e., the directory that contains the folder ".mvn".
@REM Fallback to current working directory if not found.

# 如果配置了MAVEN_BASEDIR，表示指定了打包的项目目录
set "MAVEN_PROJECTBASEDIR=%MAVEN_BASEDIR%"
if not "%MAVEN_PROJECTBASEDIR%"=="" goto endDetectBaseDir

# 如果没有指定MAVEN_BASEDIR，则开始查找打包的项目目录
# 先从当前目录开始
set "EXEC_DIR=%CD%"
set "WDIR=%EXEC_DIR%"

@REM Look for the --file switch and start the search for the .mvn directory from the specified
@REM POM location, if supplied.

# 如果命令行配置了-f other_pom.xml，即指定了pom文件选项
# 则解析这个选项
# 循环所有命令行参数，查看是否配置了 -f xx_pom.xml
set FILE_ARG=

# 查找-f参数
:arg_loop
if "%~1" == "-f" (
  # 找到
  set "FILE_ARG=%~2"
  shift
  # 处理这个参数
  goto process_file_arg
)
# 查找--file参数
if "%~1" == "--file" (
  # 找到
  set "FILE_ARG=%~2"
  shift
  # 处理这个参数
  goto process_file_arg
)
@REM If none of the above, skip the argument
shift
# 如果没有找到，还有参数，则继续找
if not "%~1" == "" (
  goto arg_loop
) else (
  # 所有参数都遍历完，也没有找到，则跳转
  goto findBaseDir
)

# 配置了-f 参数，但值为空，则跳转
:process_file_arg
if "%FILE_ARG%" == "" (
  goto findBaseDir
)

# 配置了-f 参数，但值不存在，则出错
if not exist "%FILE_ARG%" (
  echo POM file "%FILE_ARG%" specified the -f/--file command-line argument does not exist >&2
  goto error
)

# 如果配置项为目录如: -f c:/project/xxx/，则在目表示POM_DIR所在目录
if exist "%FILE_ARG%\*" (
  set "POM_DIR=%FILE_ARG%"
) else (
  # 如果配置项为文件如：-f other_pom.xml，则获取other_pom.xml文件所在目录。执行一个函数
  call :get_directory_from_file "%FILE_ARG%"
)
if not exist "%POM_DIR%" (
  echo Directory "%POM_DIR%" extracted from the -f/--file command-line argument "%FILE_ARG%" does not exist >&2
  goto error
)
set "WDIR=%POM_DIR%"
goto findBaseDir

# 函数
:get_directory_from_file
# 获取第1个参数所在目录
set "POM_DIR=%~dp1"
:stripPomDir
if not "_%POM_DIR:~-1%"=="_\" goto pomDirStripped
# 设置目录
set "POM_DIR=%POM_DIR:~0,-1%"
goto stripPomDir
# 退出函数
:pomDirStripped
exit /b
# 函数结束

# 在WDIR目录找.mvn文件夹，如果没有一直向上层找，如果找到.mvn所在目录，则项目目录为这个目录。如果没找到则使用当前目录
:findBaseDir
cd /d "%WDIR%"
:findBaseDirLoop
if exist "%WDIR%\.mvn" goto baseDirFound
cd ..
IF "%WDIR%"=="%CD%" goto baseDirNotFound
set "WDIR=%CD%"
goto findBaseDirLoop

:baseDirFound
set "MAVEN_PROJECTBASEDIR=%WDIR%"
cd /d "%EXEC_DIR%"
goto endDetectBaseDir

# 没有找到，则使用当前命令行执行的所在目录做为项目目录
:baseDirNotFound
if "_%EXEC_DIR:~-1%"=="_\" set "EXEC_DIR=%EXEC_DIR:~0,-1%"
set "MAVEN_PROJECTBASEDIR=%EXEC_DIR%"
cd "%EXEC_DIR%"

:endDetectBaseDir

# 扩展配置。如jvm参数配置
set "jvmConfig=\.mvn\jvm.config"
if not exist "%MAVEN_PROJECTBASEDIR%%jvmConfig%" goto endReadAdditionalConfig

@setlocal EnableExtensions EnableDelayedExpansion
for /F "usebackq delims=" %%a in ("%MAVEN_PROJECTBASEDIR%\.mvn\jvm.config") do set JVM_CONFIG_MAVEN_PROPS=!JVM_CONFIG_MAVEN_PROPS! %%a
@endlocal & set JVM_CONFIG_MAVEN_PROPS=%JVM_CONFIG_MAVEN_PROPS%

:endReadAdditionalConfig

# maven 项目依赖jar
for %%i in ("%MAVEN_HOME%"\boot\plexus-classworlds-*) do set CLASSWORLDS_JAR="%%i"
set CLASSWORLDS_LAUNCHER=org.codehaus.plexus.classworlds.launcher.Launcher

# 启动
"%JAVACMD%" ^
  %JVM_CONFIG_MAVEN_PROPS% ^
  %MAVEN_OPTS% ^
  %MAVEN_DEBUG_OPTS% ^
  -classpath %CLASSWORLDS_JAR% ^
  "-Dclassworlds.conf=%MAVEN_HOME%\bin\m2.conf" ^
  "-Dmaven.home=%MAVEN_HOME%" ^
  "-Dlibrary.jansi.path=%MAVEN_HOME%\lib\jansi-native" ^
  "-Dmaven.multiModuleProjectDirectory=%MAVEN_PROJECTBASEDIR%" ^
  %CLASSWORLDS_LAUNCHER% %MAVEN_CMD_LINE_ARGS%
if ERRORLEVEL 1 goto error
goto end

:error
set ERROR_CODE=1

:end
@endlocal & set ERROR_CODE=%ERROR_CODE%

# 设置脚本
if not "%MAVEN_SKIP_RC%"=="" goto skipRcPost
@REM check for post script, once with legacy .bat ending and once with .cmd ending
if exist "%USERPROFILE%\mavenrc_post.bat" call "%USERPROFILE%\mavenrc_post.bat"
if exist "%USERPROFILE%\mavenrc_post.cmd" call "%USERPROFILE%\mavenrc_post.cmd"
:skipRcPost

# 是否暂停
@REM pause the script if MAVEN_BATCH_PAUSE is set to 'on'
if "%MAVEN_BATCH_PAUSE%"=="on" pause

# 退出
if "%MAVEN_TERMINATE_CMD%"=="on" exit %ERROR_CODE%

cmd /C exit /B %ERROR_CODE%

```