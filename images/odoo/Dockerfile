FROM debian:12.6-slim AS odoo

LABEL author=danny


RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    # For Odoo
    git \
    postgresql-client \
    gettext-base \
    wget \
    # for PDF
    libssl-dev \
    fonts-cantarell \
    xfonts-75dpi \
    xfonts-base \
    # weasyprint dependencies
    libpango-1.0-0 \
    libharfbuzz0b \
    libpangoft2-1.0-0 \
    libpangocairo-1.0-0 \
    # to work inside the container
    locales \
    build-essential \
    cron \
    curl \
    vim \
    sudo \
    iputils-ping \
    watch \
    tree \
    nano \
    less \
    software-properties-common \
    bash-completion \
    # For psycopg2
    libpq-dev \
    # Other
    libffi-dev \
    liblcms2-dev \
    libldap2-dev \
    libsasl2-dev \
    libtiff5-dev \
    libwebp-dev \
    rlwrap \
    tk8.6-dev \
    ssh-client \
    # VSCode container requirements
    net-tools \
    # For pyenv build dependencies
    make \
    # For pandas
    libbz2-dev \
    # For other dependencies
    zlib1g-dev \
    libreadline-dev \
    llvm \
    libncurses5-dev \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    liblzma-dev \
    && rm -rf /var/lib/apt/lists/*

RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
    && dpkg-reconfigure --frontend=noninteractive locales

# Detect arch and install wkhtmltopdf
ENV WKHTMLTOPDF_VERSION=0.12.6.1-3
RUN if [ "$(uname -m)" = "aarch64" ]; then export ARCH=arm64; fi \
    && if [ "$(uname -m)" = "x86_64" ]; then export ARCH=amd64; fi \
    && downloaded_file=wkhtmltox_$WKHTMLTOPDF_VERSION.bookworm_${ARCH}.deb \
    && wget -q https://github.com/wkhtmltopdf/packaging/releases/download/$WKHTMLTOPDF_VERSION/$downloaded_file \
    && dpkg -i $downloaded_file \
    && rm $downloaded_file

# Create new user with home directory, improve docker compatibility with UID/GID 1000,
# add user to sudo group, allow passwordless sudo, switch to that user
# and change directory to user home directory
RUN groupadd -g 1000 odoo \
    && useradd --no-log-init -r -m -u 1000 -g 1000 -G sudo odoo \
    && echo "odoo ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

USER odoo
WORKDIR /home/odoo

# Install Python via pyenv
ENV PYTHON_VERSION=3.10.5 \
    PYENV_ROOT=/home/odoo/.pyenv \
    PATH=$PYENV_ROOT/shims:$PYENV_ROOT/bin:$PATH

# From https://github.com/pyenv/pyenv#basic-github-checkout
RUN git clone --depth 1 https://github.com/pyenv/pyenv.git .pyenv \
    && pyenv install $PYTHON_VERSION \
    && PYENV_VERSION=$PYTHON_VERSION pip install --no-cache-dir virtualenv \
    && pyenv global $PYTHON_VERSION \
    && sed -Ei -e '/^([^#]|$)/ {a export PYENV_ROOT="/home/odoo/.pyenv" a export PATH="$PYENV_ROOT/bin:$PATH" a ' -e ':a' -e '$!{n;ba};}' ~/.profile \
    && echo 'eval "$(pyenv init --path)"' >>~/.profile \
    && echo 'eval "$(pyenv init -)"' >>~/.bashrc


ENV PATH=/home/odoo/.local/bin:$PATH


# Install Node via nvm
ENV NODE_VERSION=16.18.0 \
    NVM_DIR=/home/odoo/.nvm \
    PATH=${NVM_DIR}/versions/node/v${NODE_VERSION}/bin/:${PATH}

RUN wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash \
    && . ${NVM_DIR}/nvm.sh \
    && nvm install ${NODE_VERSION} \
    && nvm use v${NODE_VERSION} \
    && npm install -g yarn \
    && nvm alias default v${NODE_VERSION} \
    && rm -rf ${NVM_DIR}/.cache \
    && echo 'export NVM_DIR="/home/odoo/.nvm"' >>~/.bashrc \
    && echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
    && echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion' >> ~/.bashrc


EXPOSE 8069 8072
