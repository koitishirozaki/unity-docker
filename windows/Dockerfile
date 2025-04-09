# Windows
ARG dlls="mcr.microsoft.com/windows/server:ltsc2022"
ARG windows="mcr.microsoft.com/dotnet/framework/sdk:4.8-windowsservercore-ltsc2022"

FROM $dlls AS dll-source
FROM $windows

SHELL ["cmd", "/C"]
COPY --from=dll-source c:/windows/system32/BluetoothApis.dll \
    c:/windows/system32/bthprops.cpl \
    c:/windows/system32/dxva2.dll \
    c:/windows/system32/mf.dll \
    c:/windows/system32/mfplat.dll \
    c:/windows/system32/mfreadwrite.dll \
    c:/windows/system32/opengl32.dll \
    c:/windows/system32/glu32.dll \
    c:/windows/system32/

RUN powershell -Command Set-ExecutionPolicy unrestricted
RUN powershell -Command Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

## Unity
ARG version
ARG module

RUN choco install unity-hub --no-progress -y
RUN choco install git --no-progress -y
RUN choco install visualstudio2022-workload-vctools --no-progress -y

SHELL [ "cmd", "/S", "/C" ]
RUN setx path "C:\\Program Files\\Git\\bin;%path%"
SHELL ["bash.exe", "-c"]

# Install Editor
RUN "mkdir -p /var/log && mkdir -p C:/UnityEditor"
RUN "C:/Program\ Files/Unity\ Hub/Unity\ Hub.exe -- --headless install-path --set C:/UnityEditor"
RUN "C:/Program\ Files/Unity\ Hub/Unity\ Hub.exe -- --headless install \
    --version $version \
    | tee /var/log/install-editor.log \
    && grep 'Error' /var/log/install-editor.log \
    | exit $(wc -l)"
RUN "if [ $module = 'base' ]; then \
    echo 'running default modules for this base OS'; \
    else \
    C:/Program\ Files/Unity\ Hub/Unity\ Hub.exe -- --headless install-modules \
    --version $version \
    --module $module \
    --childModules \
    | tee /var/log/install-module-${module}.log \
    && grep 'Missing module' /var/log/install-module-${module}.log \
    | exit $(wc -l); \
    fi"


SHELL ["powershell.exe", "-Command"]
RUN echo "$Env:version" > "C:/UnityEditor/$Env:version/version"
RUN setx -M UNITY_PATH "C:/UnityEditor/$Env:version"

# Windows services required
RUN git config --global --add safe.directory "*"
RUN foreach ("$service" in 'nlasvc', 'netprofm') {"Set-Service $service -StartupType Automatic"}

# Install modules
#########################################################
# Add Nintendo/Sony/XBOX modules here.                  #         
# In case of plugins, SDKs, etc, add them here as well  #
#########################################################
RUN if ("$Env:module" -eq 'Android') { .\scripts\Android.ps1 }

# Licenses
RUN if ("$Env:module" -eq 'Android') { .\scripts\AcceptAndroidLicenses.ps1 }

RUN Remove-Item .\scripts -Recurse -Force
