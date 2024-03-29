FROM ruby:3.0

RUN echo $(apt-cache search magick)

# Add NodeJS and Yarn repositories to apt-get
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
# Installing Node 16.x
RUN curl -sL https://deb.nodesource.com/setup_16.x | bash

# Install packages
RUN apt-get clean
RUN apt-get -qqy update \
    && apt-get install -y vim \
                          build-essential \
                          git \
                          curl \
                          locales \
                          libreadline-dev \
                          libssl-dev \
                          libsqlite3-dev \
                          wget \
                          imagemagick \
                          xz-utils \
                          libcurl4-gnutls-dev \
                          libxrender1 \
                          libfontconfig1 \
                          apt-transport-https \
                          tzdata \
                          xfonts-base \
                          xfonts-75dpi \
                          yarn \
		                      python \
                          shared-mime-info \
		                      nodejs -qqy \
    && rm -rf /var/lib/apt/lists/*

# Env variables for application
ENV DB_ADAPTER=mysql2
ENV NODE_ENV=production
ENV RAILS_SERVE_STATIC_FILES=false

# Adding project files
RUN mkdir /roadmap
COPY roadmap/ /roadmap
WORKDIR /roadmap

# Ensure its using the timezone we want
ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Set the default editor for the generation of rails credentials
RUN touch ~/.bashrc
RUN echo "export EDITOR=vim" >> ~/.bashrc

# Install Bundler
RUN gem install bundler
RUN bundle config set without 'pgsql thin rollbar'
RUN mkdir pid

# Load dependencies
RUN bundle install --jobs 20 --retry 5

# Install and run Yarn
# RUN npm install -g yarn
RUN rm -rf node_modules
RUN yarn --frozen-lockfile --production
# RUN yarn install

# Copy the custom config files for Docker
COPY overrides/config/database.yml config/database.yml
COPY overrides/config/webpacker.yml config/webpacker.yml
COPY overrides/config/environments/ci.rb config/environments/ci.rb
COPY overrides/config/initializers/dragonfly.rb config/initializers/dragonfly.rb
COPY overrides/config/initializers/wicked_pdf.rb config/initializers/wicked_pdf.rb
COPY overrides/config/webpack/ci.js config/webpack/ci.js

# Initialize the credentials file
RUN EDITOR='echo "$(cat config/credentials.yml.example)" >' bin/rails credentials:edit

# Add the wkhtmltopdf path to the ENV variables
RUN export WICKED_PDF_PATH=`which wkhtmltopdf`
RUN echo $WICKED_PDF_PATH

# Copy the startup script into the container
COPY --chown=755 overrides/docker-startup.rb ./docker-startup.rb

# expose correct ports
#   25 - email server
#   80 and 443 - HTTP traffic
#   3306 - database server
EXPOSE 25 80 443 3306

CMD ["ruby", "docker-startup.rb"]
