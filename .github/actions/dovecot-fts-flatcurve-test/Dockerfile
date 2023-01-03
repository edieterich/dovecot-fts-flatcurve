FROM debian:bullseye-slim

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get install -y \
        curl \
        gpg

RUN curl https://repo.dovecot.org/DOVECOT-REPO-GPG | gpg --import && \
        gpg --export ED409DA1 > /etc/apt/trusted.gpg.d/dovecot.gpg && \
        echo "deb https://repo.dovecot.org/ce-2.3-latest/debian/buster buster main" >> /etc/apt/sources.list.d/dovecot.list

RUN apt-get update && apt-get install -y \
	git \
	automake \
	libtool \
	wget \
	make \
	gettext \
	build-essential \
	bison \
	flex \
	valgrind \
	libssl-dev \
	pkg-config \
	libstemmer-dev \
	libexttextcat-dev \
	libicu-dev \
	libxapian-dev \
	dovecot-imaptest \
	libclucene-dev \
	unzip \
	poppler-utils \
	bash \
	procps \
	lsof \
	sudo \
	default-jre \
	libexpat1-dev

# We need to build Dovecot ourselves, since "standard" Dovecot does not
# come with necessary ICU libraries built-in
RUN mkdir /dovecot
RUN git clone --depth 1 --branch release-2.3 \
	https://github.com/dovecot/core.git /dovecot/core
COPY patches /patches
RUN cd /dovecot/core && patch -p1 < /patches/fts-squat.patch
RUN cd /dovecot/core && \
	./autogen.sh && \
	PANDOC=false ./configure --with-stemmer --with-textcat --with-icu --with-lucene --with-solr && \
	make install

RUN git clone --depth 1 https://github.com/slusarz/dovecot-fts-flatcurve.git \
	/dovecot/fts-flatcurve
RUN cd /dovecot/fts-flatcurve && ./autogen.sh && ./configure && make install

# Users dovecot and dovenull are created by dovecot-imaptest package
RUN useradd vmail && \
    mkdir -p /dovecot/sdbox && \
    chown -R vmail:vmail /dovecot/sdbox

ADD configs/ /dovecot/configs
RUN chown -R vmail:vmail /dovecot/configs/virtual
ADD imaptest/ /dovecot/imaptest

RUN wget --quiet https://www-eu.apache.org/dist/lucene/solr/8.11.2/solr-8.11.2.tgz
RUN tar xzf solr-8.11.2.tgz solr-8.11.2/bin/install_solr_service.sh --strip-components=2
RUN bash ./install_solr_service.sh solr-8.11.2.tgz && sudo -u solr /opt/solr/bin/solr create -c dovecot
RUN rm -f /var/solr/data/dovecot/conf/schema.xml
RUN rm -f /var/solr/data/dovecot/conf/managed-schema
RUN rm -f /var/solr/data/dovecot/conf/solrconfig.xml
RUN cp /dovecot/core/doc/solr-config-7.7.0.xml /var/solr/data/dovecot/conf/solrconfig.xml
RUN cp /dovecot/core/doc/solr-schema-7.7.0.xml /var/solr/data/dovecot/conf/schema.xml

ADD fts-flatcurve-test.sh /fts-flatcurve-test.sh
RUN chmod +x /fts-flatcurve-test.sh
ENTRYPOINT ["/fts-flatcurve-test.sh"]
