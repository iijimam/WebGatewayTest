FROM containers.intersystems.com/intersystems/webgateway:2022.2.0.372.0

COPY webgwfiles /etc/apache2/mods-available/webgwfiles

RUN cat /etc/apache2/mods-available/webgwfiles/webgateway.conf >> /etc/apache2/mods-available/CSP.conf &&\
    cat /etc/apache2/mods-available/webgwfiles/CSP.ini >> ${ISC_PACKAGE_INSTALLDIR}/bin/CSP.ini && \
    cp /etc/apache2/mods-available/webgwfiles/server.crt /etc/ssl/certs/server.crt && \
    cp /etc/apache2/mods-available/webgwfiles/server.key /etc/ssl/private/server.key && \
    cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/default-ssl-bck.conf && \
    sed -e 's/ssl-cert-snakeoil.pem/server.crt/g' /etc/apache2/sites-available/default-ssl.conf > /etc/apache2/sites-available/default-ssl2.conf && \
    sed -e 's/ssl-cert-snakeoil.key/server.key/g' /etc/apache2/sites-available/default-ssl2.conf > /etc/apache2/sites-available/default-ssl.conf && \
    rm /etc/apache2/sites-available/default-ssl2.conf && \
    a2ensite default-ssl && \
    a2enmod ssl


