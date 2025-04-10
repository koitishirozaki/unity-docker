ARG baseImage="base:windows"
ARG hubImage="unityhub:windows"

FROM $hubImage AS Builder
RUN choco install git --no-progress -y

SHELL [ "cmd", "/S", "/C" ]
RUN setx path "C:\\Program Files\\Git\\bin;%path%"

SHELL ["bash.exe", "-c"]

ARG version
ARG module=base
ARG changeset

# Install Editor
RUN "mkdir -p C:/logs && mkdir -p C:/Unity && mkdir -p C:/ext"
RUN "C:/Program\ Files/Unity\ Hub/Unity\ Hub.exe -- --headless install-path --set C:/Unity"
RUN echo "Version: $version" > C:/logs/version.log

RUN "C:/Program\ Files/Unity\ Hub/Unity\ Hub.exe -- --headless install \
    --version ${version}\
    --changeset ${changeset}\
    | tee C:/logs/install-editor.log \
    && grep 'Error' C:/logs/install-editor.log \
    | exit $(wc -l)"

# # Installing modules dependencies
RUN "if [ $module = 'base' ]; then \
    echo 'running default modules for this base OS'; \
    else \
    echo 'running custom modules'; \
    C:/Program\ Files/Unity\ Hub/Unity\ Hub.exe -- --headless install-modules \
    --version ${version} \
    --changeset ${changeset}\
    --module ${module} \
    --childModules \
    | tee C:/logs/install-module-${module}.log \
    && grep 'Missing module' C:/logs/install-module-${module}.log \
    | exit $(wc -l); \
    fi"


# # #########################################################
# # # Add Nintendo/Sony/XBOX modules here.                  #         
# # # In case of plugins, SDKs, etc, add them here as well  #
# # #########################################################
FROM ${baseImage}

SHELL ["powershell.exe", "-Command"]
# Need to grab these dependencies that the editor needs to run
COPY --from=Builder ["C:/Unity/", "C:/Unity/"]

COPY --from=Builder C:/windows/system32/MSVCP100.dll \
    C:/windows/system32/MSVCP120.dll \
    C:/windows/system32/MSVCR100.dll \
    C:/windows/system32/MSVCR120.dll \
    C:/windows/system32/

ARG version
RUN echo "$Env:version" > "C:/Unity/$Env:version/version"

RUN setx -M UNITY_PATH "C:/Unity/$version"

RUN choco install git --no-progress -y
RUN git config --global --add safe.directory "*"
RUN foreach ("$service" in 'nlasvc', 'netprofm') {"Set-Service $service -StartupType Automatic"}

# # Set Unity Target Platform
# RUN if ("$Env:module" -eq 'Android') { .\scripts\SetupAndroid.ps1 }

# # Licenses
# RUN if ("$Env:module" -eq 'Android') { .\scripts\AcceptAndroidLicenses.ps1 }

# RUN Remove-Item .\scripts -Recurse -Force
RUN choco install visualstudio2022-workload-vctools --no-progress -y
