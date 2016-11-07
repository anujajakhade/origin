#
# This is the HAProxy router for OpenShift Origin.
#
# The standard name for this image is openshift/origin-haproxy-router
#
FROM openshift/origin

#
# Note: /var is changed to 777 to allow access when running this container as a non-root uid
#       this is temporary and should be removed when the container is switch to an empty-dir
#       with gid support.
#

RUN yum install --nogpgcheck -y \
        git \
        java-1.8.0-openjdk \
        gcc-c++ \
        tar \
        openssl \
        openssl-devel \
        pcre \
        pcre-devel \
        make
RUN yum clean all

# Clone the source code of HAProxy from github
#RUN git clone http://git.haproxy.org/git/haproxy-1.6.git
RUN wget http://www.haproxy.org/download/1.6/src/haproxy-1.6.9.tar.gz && tar xzvf haproxy-1.6.9.tar.gz

# Build and install Node.js
RUN cd haproxy-1.6.9 && make TARGET=linux26 USE_OPENSSL=1 && make install

RUN  mkdir -p /var/lib/haproxy/{conf,run,bin,log} && \
    touch /var/lib/haproxy/conf/{{os_http_be,os_edge_http_be,os_tcp_be,os_sni_passthrough,os_reencrypt,os_edge_http_expose,os_edge_http_redirect}.map,haproxy.config} && \
    chmod -R 777 /var && \
setcap 'cap_net_bind_service=ep' /usr/local/sbin/haproxy

COPY . /var/lib/haproxy/

LABEL io.k8s.display-name="OpenShift Origin HAProxy Router" \
      io.k8s.description="This is a component of OpenShift Origin and contains an HAProxy instance that automatically exposes services within the cluster through routes, and offers TLS termination, reencryption, or SNI-passthrough on ports 80 and 443."
USER 1001
EXPOSE 80 443
WORKDIR /var/lib/haproxy/conf
ENV TEMPLATE_FILE=/var/lib/haproxy/conf/haproxy-config.template \
    RELOAD_SCRIPT=/var/lib/haproxy/reload-haproxy
ENTRYPOINT ["/usr/bin/openshift-router"]
