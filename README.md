# ModSecurity
## ModSecurity Setup [apache] Reverse proxy

1. apt-get install apache2
2. mkdir /var/log/modsecurity #save the modsecurity audit log
3. mkdir /opt/modsecurity #dir to install the modsecurity
4. cd /opt/modsecurity 
5. wget https://www.modsecurity.org/tarball/2.9.3/modsecurity-2.9.3.tar.gz
6. tar -xvf modsecurity-2.9.3.tar.gz
7. apt-get install g++ flex bison curl doxygen libyajl-dev libgeoip-dev libtool dh-autoreconf libcurl4-gnutls-dev libxml2 libpcre++-dev libxml2-dev 
8. apt-get install libyajl-dev
9. ./autogen.sh
10. ./configure
11. make
12. make install
13. cp /usr/local/modsecurity/lib/mod_security2.so /usr/lib/apache2/modules #copying security module to apache module dir
14. nano /etc/apache2/mods-available/security2.conf

'''

     <IfModule security2_module>
     #Default Debian dir for modsecurity's persistent data
     SecDataDir /var/cache/modsecurity
     #Include all the *.conf files in /etc/modsecurity.
     #Keeping your local configuration in that directory
     #will allow for an easy upgrade of THIS file and
     #make your life easier
     IncludeOptional /opt/modsecurity/*.conf  #change the dir location based on your modsecurity install
     </IfModule>

'''

15. nano /etc/apache2/mods-available/security2.load

#Depends: unique_id
LoadFile libxml2.so.2
LoadModule security2_module /usr/lib/apache2/modules/mod_security2.so

16. a2enmod security2
17. a2enmod http_proxy
18. service apache2 start 

Now check in apache error.log to validate the ModSecurity.  

Setting up OWASP CRS rules in ModSecurity

1. cd /etc/apache/modsecurity
2. wget https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/v3.2.0.tar.gz # url https://coreruleset.org/ to download
3. tar -xvf v3.2.0.tar.gz
4. cd <unpacked dir>
5. rename crs-setup.conf.example to crs-setup.conf | mv /etc/apache/modsecurity/<dir>/rs-setup.conf.example /etc/apache/modsecurity/<dir>/rs-setup.conf
6. Rename rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example and
    rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example to remove the
    '.example' extension. This will allow you to add exclusions without updates
    overwriting them in the future.
7. Add the following line to your httpd.conf/apache2.conf (the following
    assumes you've cloned CRS into modsecurity.d/owasp-modsecurity-crs). You
    can alternatively place these in any config file included by Apache:

    <IfModule security2_module>
                Include modsecurity.d/owasp-modsecurity-crs/crs-setup.conf
                Include modsecurity.d/owasp-modsecurity-crs/rules/*.conf
    </IfModule>
8. Restart web server

Apache2 Reverse Proxy
---------------------

Add the below line in /etc/apache2/site-available/000-default.conf

     ProxyPass / http://10.138.114.231:7070/   # Based on your web server IP and Port
     ProxyPassReverse / http://10.138.114.231:7070/ # Based on your web server IP and Port
     ProxyPreserveHost On # Preserve the Host header 
 
 










