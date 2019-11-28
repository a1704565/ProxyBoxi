# ProxyBoxi



# Squid:n asennus ProxiBoxi:n

Ensimmäiseksi pitää tehdä laajennettu ext osio squidin cache-muistille jossa on vähintään 600 Mb tilaa, ja tähän pitää viitata /tmp/squid/.



# Squid configuraatio

acl localnet src 10.0.0.0/8 

acl localnet src 172.16.0.0/12 

acl localnet src 192.168.0.0/16 

acl localnet src fc00::/7 

acl localnet src 192.168.1.111/16 

acl localnet src fe80::/10 

  

acl ssl_ports port 443 

  

acl safe_ports port 80 

acl safe_ports port 21 

acl safe_ports port 443 

acl safe_ports port 70 

acl safe_ports port 210 

acl safe_ports port 1025-65535 

acl safe_ports port 280 

acl safe_ports port 488 

acl safe_ports port 591 

acl safe_ports port 777 

acl connect method connect 

acl blocksites dstdomain "/etc/squid/Ads.conf" 

  

http_access deny !safe_ports 

http_access deny connect !ssl_ports 

  

http_access deny localhost manager 

http_access deny manager 

  

http_access deny to_localhost 

  

http_access allow localnet 

http_access allow localhost 

  

http_access deny all 

  

refresh_pattern ^ftp: 1440 20% 10080 

refresh_pattern ^gopher: 1440 0% 1440 

refresh_pattern -i (/cgi-bin/|\?) 0 0% 0 

refresh_pattern . 0 20% 4320 

  

access_log none 

cache_log /dev/null 

cache_store_log stdio:/dev/null 

logfile_rotate 0 

  

logfile_daemon /dev/null 

  

http_port 3128 intercept 

  

# cache_dir aufs Directory-Name Mbytes L1 L2 [options] 

cache_dir aufs /tmp/squid/cache 600 16 512 

  

# If you have 64 MB device RAM you can use 16 MB cache_mem, default is 8 MB 

cache_mem 8 MB              

maximum_object_size_in_memory 100 KB 

maximum_object_size 32 MB 
