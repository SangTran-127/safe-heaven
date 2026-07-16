# Use the official Microsoft Go devcontainer base image
FROM mcr.microsoft.com/devcontainers/go:1-bookworm

# Set custom username argument (e.g. sang)
ARG NEW_USER=sang

# Rename default user 'vscode' to NEW_USER, rename home directory and update permissions
USER root
RUN usermod -l ${NEW_USER} vscode \
    && usermod -d /home/${NEW_USER} -m ${NEW_USER} \
    && groupmod -n ${NEW_USER} vscode \
    && echo "${NEW_USER} ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/${NEW_USER} \
    && chmod 0440 /etc/sudoers.d/${NEW_USER} \
    && rm -f /etc/sudoers.d/vscode

# Switch to the newly renamed non-root user
USER ${NEW_USER}

# Configure Git to trust all directories to avoid dubious ownership errors on volume mounts
RUN git config --global --add safe.directory "*"