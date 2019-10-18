---


---

<h1 id="tp3">TP3</h1>
<p><strong>1. Config du resolver dns du système:</strong></p>
<pre><code>vagrant@proxy:~$ cat /etc/resolv.conf 
nameserver 192.168.33.21
nameserver 192.168.33.22
</code></pre>
<p><strong>2. Trouver l’adresse ip de l’host de wiki.lab.local:</strong></p>
<pre><code>vagrant@proxy:/$ dig wiki.lab.local +short
192.168.56.11
</code></pre>
<p><strong>3. Services réseaux qui sont en fonctionnement actuellement sur auth-1:</strong></p>
<pre><code>[vagrant@auth-1 /]$ ss -lnut
Netid  State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
udp    UNCONN     0      0                *:960                          *:*                  
udp    UNCONN     0      0                *:17642                        *:*                  
udp    UNCONN     0      0                *:53                           *:*                  
udp    UNCONN     0      0        127.0.0.1:323                          *:*                  
udp    UNCONN     0      0                *:68                           *:*                  
udp    UNCONN     0      0                *:111                          *:*                  
udp    UNCONN     0      0               :::960                         :::*                  
udp    UNCONN     0      0               :::15297                       :::*                  
udp    UNCONN     0      0               :::53                          :::*                  
udp    UNCONN     0      0              ::1:323                         :::*                  
udp    UNCONN     0      0               :::111                         :::*                  
tcp    LISTEN     0      50       127.0.0.1:3306                         *:*                  
tcp    LISTEN     0      128              *:111                          *:*                  
tcp    LISTEN     0      128    192.168.33.31:80                           *:*                  
tcp    LISTEN     0      128              *:53                           *:*                  
tcp    LISTEN     0      128              *:22                           *:*                  
tcp    LISTEN     0      100      127.0.0.1:25                           *:*                  
tcp    LISTEN     0      128             :::111                         :::*                  
tcp    LISTEN     0      128             :::53                          :::*                  
tcp    LISTEN     0      128             :::22                          :::*                  
tcp    LISTEN     0      100            ::1:25                          :::*     
</code></pre>
<p><strong>4. Services réseaux qui sont en fonctionnement actuellement sur recursor-1:</strong></p>
<pre><code>[vagrant@recursor-1 ~]$ ss -lut
Netid  State      Recv-Q Send-Q Local Address:Port                 Peer Address:Port                
udp    UNCONN     0      0              *:960                          *:*                    
udp    UNCONN     0      0      192.168.33.21:domain                       *:*                    
udp    UNCONN     0      0      127.0.0.1:domain                       *:*                    
udp    UNCONN     0      0      127.0.0.1:323                          *:*                    
udp    UNCONN     0      0              *:bootpc                       *:*                    
udp    UNCONN     0      0              *:sunrpc                       *:*                    
udp    UNCONN     0      0             :::960                         :::*                    
udp    UNCONN     0      0            ::1:323                         :::*                    
udp    UNCONN     0      0             :::sunrpc                      :::*                    
tcp    LISTEN     0      128            *:sunrpc                       *:*                    
tcp    LISTEN     0      128    192.168.33.21:domain                       *:*                    
tcp    LISTEN     0      128    127.0.0.1:domain                       *:*                    
tcp    LISTEN     0      128            *:ssh                          *:*                    
tcp    LISTEN     0      100    127.0.0.1:smtp                         *:*                    
tcp    LISTEN     0      128           :::sunrpc                      :::*                    
tcp    LISTEN     0      128           :::ssh                         :::*                    
tcp    LISTEN     0      100          ::1:smtp                        :::*    
</code></pre>
<ul>
<li>Avec la commande <code>ss -lutn</code> pour avoir les numéro au lieu des noms:</li>
</ul>
<pre><code>[vagrant@recursor-1 ~]$ ss -lutn
Netid  State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
udp    UNCONN     0      0                *:960                          *:*                  
udp    UNCONN     0      0      192.168.33.21:53                           *:*                  
udp    UNCONN     0      0        127.0.0.1:53                           *:*                  
udp    UNCONN     0      0        127.0.0.1:323                          *:*                  
udp    UNCONN     0      0                *:68                           *:*                  
udp    UNCONN     0      0                *:111                          *:*                  
udp    UNCONN     0      0               :::960                         :::*                  
udp    UNCONN     0      0              ::1:323                         :::*                  
udp    UNCONN     0      0               :::111                         :::*                  
tcp    LISTEN     0      128              *:111                          *:*                  
tcp    LISTEN     0      128    192.168.33.21:53                           *:*                  
tcp    LISTEN     0      128      127.0.0.1:53                           *:*                  
tcp    LISTEN     0      128              *:22                           *:*                  
tcp    LISTEN     0      100      127.0.0.1:25                           *:*                  
tcp    LISTEN     0      128             :::111                         :::*                  
tcp    LISTEN     0      128             :::22                          :::*                  
tcp    LISTEN     0      100            ::1:25                          :::*                  
</code></pre>
<p><strong>5. Où sont configuré chacun de ces composants ?</strong></p>
<pre><code>/etc/pdns-recursor/recursor.conf
/etc/NetworkManager/NetworkManager.conf
/etc/resolv.conf
</code></pre>
<p><strong>6. Qu’est ce qui est configuré sur les serveurs recursifs pour le domaine lab.local ? (Important pour les Actions qui suivent)</strong></p>
<ul>
<li>Du forwarding vers les serveurs autoritatif !!</li>
</ul>
<pre><code>[vagrant@recursor-1 ~]$ cat /etc/pdns-recursor/recursor.conf | grep lab.local 
forward-zones=lab.local=192.168.56.31;192.168.56.32
</code></pre>
<p><strong>7. Comment mettre en évidence le fait que le récurseurs ne répondent que sur l’interface interne ?</strong></p>
<pre><code>[idk@localhost blog]$ dig wiki.lab.local @192.168.56.21 +short
</code></pre>
<pre><code>[vagrant@recursor-1 ~]$ ss -utpl
Netid  State      Recv-Q Send-Q Local Address:Port                 Peer Address:Port                
udp    UNCONN     0      0              *:960                          *:*                    
udp    UNCONN     0      0      192.168.33.21:domain                       *:*                    
udp    UNCONN     0      0      127.0.0.1:domain                       *:*                    
udp    UNCONN     0      0      127.0.0.1:323                          *:*                    
udp    UNCONN     0      0              *:bootpc                       *:*                    
udp    UNCONN     0      0              *:sunrpc                       *:*                    
udp    UNCONN     0      0             :::960                         :::*                    
udp    UNCONN     0      0            ::1:323                         :::*                    
udp    UNCONN     0      0             :::sunrpc                      :::*                    
tcp    LISTEN     0      128            *:sunrpc                       *:*                    
tcp    LISTEN     0      128    192.168.33.21:domain                       *:*                    
tcp    LISTEN     0      128    127.0.0.1:domain                       *:*                    
tcp    LISTEN     0      128            *:ssh                          *:*                    
tcp    LISTEN     0      100    127.0.0.1:smtp                         *:*                    
tcp    LISTEN     0      128           :::sunrpc                      :::*                    
tcp    LISTEN     0      128           :::ssh                         :::*                    
tcp    LISTEN     0      100          ::1:smtp                        :::*     
</code></pre>
<p><strong>8. Comment est sécurisé l’accès à mysql ?</strong><br>
3306 est le port mysql (Ouais je connais deux trois port par coeur)<br>
ON voit le service mysql n’écoute que sur l’interface 127.0.0.1 (localhost) sur le port 3306</p>
<pre><code>[vagrant@auth-1 /]$ ss -lnut
Netid  State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
udp    UNCONN     0      0                *:960                          *:*                  
udp    UNCONN     0      0                *:17642                        *:*                  
udp    UNCONN     0      0                *:53                           *:*                  
udp    UNCONN     0      0        127.0.0.1:323                          *:*                  
udp    UNCONN     0      0                *:68                           *:*                  
udp    UNCONN     0      0                *:111                          *:*                  
udp    UNCONN     0      0               :::960                         :::*                  
udp    UNCONN     0      0               :::15297                       :::*                  
udp    UNCONN     0      0               :::53                          :::*                  
udp    UNCONN     0      0              ::1:323                         :::*                  
udp    UNCONN     0      0               :::111                         :::*                  
tcp    LISTEN     0      50       127.0.0.1:3306                         *:*                  
tcp    LISTEN     0      128              *:111                          *:*                  
tcp    LISTEN     0      128    192.168.33.31:80                           *:*                  
tcp    LISTEN     0      128              *:53                           *:*                  
tcp    LISTEN     0      128              *:22                           *:*                  
tcp    LISTEN     0      100      127.0.0.1:25                           *:*                  
tcp    LISTEN     0      128             :::111                         :::*                  
tcp    LISTEN     0      128             :::53                          :::*                  
tcp    LISTEN     0      128             :::22                          :::*                  
tcp    LISTEN     0      100            ::1:25                          :::*     

</code></pre>

