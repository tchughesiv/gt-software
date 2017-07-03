FROM registry.access.redhat.com/rhel7

LABEL name="gt-software/ftp" \
      vendor="gt-software" \
      version="1.0.42" \
      release="1" \
      summary="Acme Corp's Starter app" \
      description="Starter app will do ....." \
### Required labels above - recommended below
      url="https://www.acme.io" \
      run='docker run -tdi --name ${NAME} ${IMAGE}' \
      io.k8s.description="Starter app will do ....." \
      io.k8s.display-name="Starter app" \
      io.openshift.expose-services="" \
      io.openshift.tags="acme,starter"

ENV FTP_DIR=/var/ivory \
    FTP_USER=ivory \
    FTP_PASS=ivory \
    FTP_UID=10001

COPY help.md /tmp/
COPY licenses /licenses

RUN REPOLIST=rhel-7-server-rpms,rhel-7-server-optional-rpms,epel && \
    yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm && \
    yum -y install --disablerepo "*" --enablerepo ${REPOLIST} --setopt=tsflags=nodocs \
      golang-github-cpuguy83-go-md2man ftp rpm-build redhat-rpm-config make gcc && \
    yum-builddep -y --disablerepo "*" --enablerepo ${REPOLIST} --setopt=tsflags=nodocs pure-ftpd && \
    mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS} && \
    echo '%_topdir %(echo $HOME)/rpmbuild' > ~/.rpmmacros && \
    cd /tmp && \
    yumdownloader --disablerepo "*" --enablerepo ${REPOLIST} --source pure-ftpd && \
    rpm -i pure-ftpd-*.src.rpm && \
    cd ~/rpmbuild/SPECS && \
    sed -i '/^%configure/a\            --with-nonroot \\'  pure-ftpd.spec && \
    sed -i 's/--with-capabilities/--without-capabilities/g' pure-ftpd.spec && \
    rpmbuild -ba pure-ftpd.spec && \
    yum -y install --disablerepo "*" --enablerepo ${REPOLIST} --setopt=tsflags=nodocs \
      ~/rpmbuild/RPMS/x86_64/pure-ftpd-*.x86_64.rpm && \
    go-md2man -in /tmp/help.md -out /help.1 && \
    yum -y autoremove rpm-build redhat-rpm-config make gcc golang-github-cpuguy83-go-md2man && \
    rm -rf ~/rpmbuild /tmp/pure-ftpd-*.src.rpm && \
    yum clean all

RUN mkdir -p ${FTP_DIR} && \
    useradd -l -u ${FTP_UID} -r -g 0 -s /sbin/nologin -c "${FTP_USER}" ${FTP_USER} && \
    echo ${FTP_USER}:${FTP_PASS} | chpasswd -c SHA512 && \
    chown -R ${FTP_UID}:0 ${FTP_DIR} /etc/pure-ftpd && \
    chmod -R g=u ${FTP_DIR} /etc/pure-ftpd

EXPOSE 2121 30000-30009

USER 10001
VOLUME ${FTP_DIR}
CMD pure-ftpd -c 50 -C 10 -l unix -E -j -R -P ${PUBLICHOST} -p 30000:30009
