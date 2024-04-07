# Regra I: Libere qualquer tráfego para interface de loopback
``iptables -A INPUT -i lo -j ACCEPT``

# Regra II: Estabeleça a política DROP para as chains INPUT e FORWARD
``iptables -P INPUT DROP``

``iptables -P FORWARD DROP``

# Regra III: Possibilite acesso ao serviço WWW (HTTP e HTTPS) para a rede interna e faça NAT
``iptables -A FORWARD -i eth0 -o eth2 -s 10.1.1.0/24 -p tcp --dport 80 -j ACCEPT``

``iptables -A FORWARD -i eth0 -o eth2 -s 10.1.1.0/24 -p tcp --dport 443 -j ACCEPT``

``iptables -t nat -A POSTROUTING -o eth2 -s 10.1.1.0/24 -j MASQUERADE``

# Regra IV: Faça LOG e bloqueie acesso a sites com a palavra "games"
``iptables -A FORWARD -m string --string "games" --algo kmp --to 65535 -j LOG --log-prefix "Blocked Games: "``

``iptables -A FORWARD -m string --string "games" --algo kmp --to 65535 -j DROP``

# Regra V: Bloqueie acesso para todos ao site www.jogosonline.com.br, exceto para o chefe
``iptables -A FORWARD -d www.jogosonline.com.br -j DROP``

``iptables -A FORWARD -s 10.1.1.100 -d www.jogosonline.com.br -j ACCEPT``

# Regra VI: Permita ICMP echo-request com limitação de 5 pacotes por segundo
``iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 5/s -j ACCEPT``

# Regra VII: Permita consultas DNS externas
``iptables -A FORWARD -i eth0 -o eth2 -p udp --dport 53 -j ACCEPT``

``iptables -A FORWARD -i eth1 -o eth2 -p udp --dport 53 -j ACCEPT``

# Regra VIII: Permita tráfego TCP destinado à máquina 192.168.1.100 (DMZ) na porta 80, vindo de qualquer rede
``iptables -A FORWARD -d 192.168.1.100 -p tcp --dport 80 -j ACCEPT``

# Regra IX: Redirecione pacotes TCP destinados ao IP 200.20.5.1 porta 80, para a máquina 192.168.1.100 na DMZ
``iptables -t nat -A PREROUTING -i eth2 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100``

# Regra X: Permita que a máquina 192.168.1.100 responda corretamente aos pacotes TCP recebidos na porta 80
``iptables -A FORWARD -s 192.168.1.100 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT``
