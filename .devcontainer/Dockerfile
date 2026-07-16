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

# Install Powerlevel10k theme and plugins (autosuggestions & syntax-highlighting)
RUN git clone --depth=1 https://github.com/romkatv/powerlevel10k.git /home/${NEW_USER}/.oh-my-zsh/custom/themes/powerlevel10k \
    && git clone https://github.com/zsh-users/zsh-autosuggestions /home/${NEW_USER}/.oh-my-zsh/custom/plugins/zsh-autosuggestions \
    && git clone https://github.com/zsh-users/zsh-syntax-highlighting.git /home/${NEW_USER}/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting

# Configure .zshrc theme and plugins
RUN sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="powerlevel10k\/powerlevel10k"/' /home/${NEW_USER}/.zshrc \
    && sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions zsh-syntax-highlighting)/' /home/${NEW_USER}/.zshrc

# Configure Git to trust all directories to avoid dubious ownership errors on volume mounts
RUN git config --global --add safe.directory "*"