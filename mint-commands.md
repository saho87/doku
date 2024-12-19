# Software und Repositorys

```bash
apt update # Aktualisiert die Paketdatenbank (keine Änderungen an installierten Paketen)
apt upgrade # Installiert neue Versionen der Pakete basierend auf den aktualisierten Paketlisten
apt list -a kubeadm # Anzeige aller verfügbaren Pakete, die in Repos verfügbar sind
apt upgrade -y kubeadm=1.31.0-1.1 # Upgrade von Paket auf bestimmte Version
sudo apt-mark hold kubeadm # kubeadm ald Held Package markieren, dass nicht automatisch bei apt upgrade aktualisiert wird 

```