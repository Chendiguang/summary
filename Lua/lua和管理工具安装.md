# 安装

安装```Lua```及其管理工具```luarocks```

Linux的安装教程官方已经详细给出，而且网上的教程较多，这里就不多做论述了。这里主要基于[这篇博客](https://blog.csdn.net/techfield/article/details/82896403), 以Windows作为环境进行安装。

* 安装配置 [MinGW](https://nuwen.net/files/mingw/mingw-16.0.exe), 根据步骤安装即可，注意```配置环境变量```。

    验证：where gcc

* 编译安装Lua

    1. 下载

        到[官方地址](http://www.lua.org/ftp/ )下载lua-5.3.5.tar.gz, 解压到c:/local/

    2. 编写build.bat 以管理员的身份执行。进行编译安装，并且配置环境变量。最后需要手动添加环境变量```C:\local\lua-5.3.5\dist\b```

        ```bat
        @echo off
        :: ========================
        :: build.bat
        ::
        :: build lua to dist folder
        :: tested with lua-5.3.5
        :: based on:
        :: https://medium.com/@CassiusBenard/lua-basics-windows-7-installation-and-running-lua-files-from-the-command-line-e8196e988d71
        :: ========================
        setlocal
        :: you may change the following variable’s value
        :: to suit the downloaded version
        set work_dir=%~dp0
        :: Removes trailing backslash
        :: to enhance readability in the following steps
        set work_dir=%work_dir:~0,-1%
        set lua_install_dir=%work_dir%\dist
        set compiler_bin_dir=%work_dir%\tdm-gcc\bin
        set lua_build_dir=%work_dir%
        set path=%compiler_bin_dir%;%path%
        cd /D %lua_build_dir%
        make PLAT=mingw
        echo.
        echo **** COMPILATION TERMINATED ****
        echo.
        echo **** BUILDING BINARY DISTRIBUTION ****
        echo.
        :: create a clean “binary” installation
        mkdir %lua_install_dir%
        mkdir %lua_install_dir%\doc
        mkdir %lua_install_dir%\bin
        mkdir %lua_install_dir%\include
        mkdir %lua_install_dir%\lib
        copy %lua_build_dir%\doc\*.* %lua_install_dir%\doc\*.*
        copy %lua_build_dir%\src\*.exe %lua_install_dir%\bin\*.*
        copy %lua_build_dir%\src\*.dll %lua_install_dir%\bin\*.*
        copy %lua_build_dir%\src\luaconf.h %lua_install_dir%\include\*.*
        copy %lua_build_dir%\src\lua.h %lua_install_dir%\include\*.*
        copy %lua_build_dir%\src\lualib.h %lua_install_dir%\include\*.*
        copy %lua_build_dir%\src\lauxlib.h %lua_install_dir%\include\*.*
        copy %lua_build_dir%\src\lua.hpp %lua_install_dir%\include\*.*
        copy %lua_build_dir%\src\liblua.a %lua_install_dir%\lib\liblua.a
        echo.
        echo **** BINARY DISTRIBUTION BUILT ****
        echo.
        %lua_install_dir%\bin\lua.exe -e "print [[Hello!]];print[[Simple Lua test successful!!!]]"
        echo.

        :: configure environment variable
        :: https://stackoverflow.com/a/21606502/4394850
        :: http://lua-users.org/wiki/LuaRocksConfig
        :: SETX - Set an environment variable permanently.
        :: /m Set the variable in the system environment HKLM.
        setx LUA "%lua_install_dir%\bin\lua.exe" /m
        setx LUA_BINDIR "%lua_install_dir%\bin" /m
        setx LUA_INCDIR "%lua_install_dir%\include" /m
        setx LUA_LIBDIR "%lua_install_dir%\lib" /m

        pause

        ```

* 安装Luarocks

    1. 下载

        从[官方列表](http://luarocks.github.io/luarocks/releases/)选择合适的版本。这里选择luarocks-3.2.1.tar.gz, 解压到c:/lcoal

    2. 安装

        注意添加环境变量：C:\local\LuaRocks-3.2.1

        ```bash
        ./install.bat /F /MW /LUA C:/local/lua-5.3.5/dist /P C:/local/LuaRocks-3.2.1 /NOADMIN /SELFCONTAINED /Q
        ```

    3. 配置

        主要是修改system 下面的root的值为```C:\local\lua-5.3.5\dist\```，以便可以找到用luarocks安装的包。

        注意：C:\local\lua-5.3.5\dist\bin这个也能安装，但是调用时失败，这跟lua查找机制相关。

        ```lua
        rocks_trees = {
            { name = [[user]],
                root    = home..[[/luarocks]],
            },
            { name = [[system]],
                --  root    = [[C:\local\LuaRocks-3.2.1\systree]],
                root    = [[C:\local\lua-5.3.5\dist\]],
            },
        }
        variables = {
            MSVCRT = 'm',   -- make MinGW use MSVCRT.DLL as runtime
            LUALIB = 'lua53.dll',
            CC = [[D:\software\mingw64\bin\gcc.exe]],
            MAKE = [[D:\software\mingw64\bin\make.exe]],
            RC = [[D:\software\mingw64\bin\windres.exe]],
            LD = [[D:\software\mingw64\bin\gcc.exe]],
            AR = [[D:\software\mingw64\bin\ar.exe]],
            RANLIB = [[D:\software\mingw64\bin\ranlib.exe]],
        }
        verbose = false   -- set to 'true' to enable verbose output

        ````

    4. 验证

        ```bash
        where luarocks

        # 尝试安装lfs
        luarocks install luafilesystem
        ```
