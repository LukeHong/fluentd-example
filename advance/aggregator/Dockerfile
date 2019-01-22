FROM fluent/fluentd:v1.3-onbuild-1

# Use root account to use apk
USER root

# below RUN includes plugin as examples elasticsearch is not required
# you may customize including plugins as you wish
RUN apk add --no-cache --update --virtual .build-deps \
        sudo build-base ruby-dev \
 && sudo gem install --no-document \
        fluent-plugin-gcs -v "0.4.0.beta1" \
 && sudo gem sources --clear-all \
 && apk del .build-deps \
 && rm -rf /home/fluent/.gem/ruby/2.5.0/cache/*.gem

USER fluent

RUN mkdir -p /fluentd/nginx/ && mkdir /fluentd/log/buffer
COPY fluentd-gcs.json /fluentd/etc