FROM mcr.microsoft.com/vscode/devcontainers/universal:latest

# Copy custom first notice message.
COPY first-run-notice.txt /tmp/staging/
RUN sudo mv -f /tmp/staging/first-run-notice.txt /usr/local/etc/vscode-dev-containers/ \
    && sudo rm -rf /tmp/staging

# Install Node.js and npm
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash - \
    && sudo apt-get install -y nodejs

# Install Azure Functions Core Tools
RUN sudo npm install -g azure-functions-core-tools

# Install .NET 8 runtime required by the PowerShell language worker.
# Without it, `func start` fails with:
#   Microsoft.NETCore.App, version '8.0.0' (x64) was not found.
# The universal image ships newer runtimes (9.0/10.0) but not 8.0 specifically.
#
# TODO: Bump to .NET 10 once the Azure Functions PowerShell worker ships
# support for PowerShell 7.6 (which targets .NET 10). As of June 2026 the
# worker is pinned to PowerShell 7.4 + .NET 8 LTS (EOL Nov 10, 2026). Track:
#   https://learn.microsoft.com/azure/azure-functions/functions-versions
RUN curl -fsSL https://packages.microsoft.com/config/ubuntu/$(. /etc/os-release && echo $VERSION_ID)/packages-microsoft-prod.deb -o /tmp/packages-microsoft-prod.deb \
    && sudo dpkg -i /tmp/packages-microsoft-prod.deb \
    && rm /tmp/packages-microsoft-prod.deb \
    && sudo apt-get update \
    && sudo apt-get install -y aspnetcore-runtime-8.0

# Install azurite to emulate Azure Storage
RUN sudo npm install -g azurite

# Install Azure Developer CLI
RUN curl -fsSL https://aka.ms/install-azd.sh | bash
