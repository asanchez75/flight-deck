FROM solr:5.5-alpine
MAINTAINER tess@ten7.com

USER root

RUN apk add --update --no-cache \
      ansible \
      bash \
      rsync

ADD http://ftp.drupal.org/files/projects/search_api_solr-8.x-1.x-dev.tar.gz /opt/search_api_solr.tar.gz
RUN tar -xzf /opt/search_api_solr.tar.gz -C /opt/

# Copy various key scripts to a directory in $PATH.
COPY docker-entrypoint.sh /usr/local/bin/

# Make them all executable.
RUN chmod a+x /usr/local/bin/docker-entrypoint.sh

# Copy the entire contents of the init.d directory.
COPY docker-entrypoint-init.d /docker-entrypoint-init.d
RUN chmod a+x -R /docker-entrypoint-init.d

USER $SOLR_USER

EXPOSE 8983

# Set the entrypoint and default command.
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
CMD ["solr-foreground"]
