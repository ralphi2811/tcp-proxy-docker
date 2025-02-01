# tcp-proxy-docker

## Description

Un utilitaire simple pour rediriger le trafic TCP vers un hôte et un port spécifiques.

Particulièrement utile pour :
- Exposer des services internes (comme des TPE) vers Internet de manière sécurisée
- Rediriger des ports d'une machine virtuelle vers des équipements sur différents réseaux internes
- Créer des tunnels TCP pour accéder à des services isolés

## Prérequis

- Docker installé sur votre machine
- Accès aux ports réseau sur votre machine hôte
- Les équipements cibles doivent être accessibles depuis la machine hôte

## Installation

### Option 1 : Utilisation de l'image Docker Hub

L'image Docker est directement utilisable depuis Docker Hub :

```bash
docker pull ralphi2811/tcp-proxy:latest
```

### Option 2 : Construction de l'image

Vous pouvez également construire l'image vous-même depuis les sources :

1. Clonez le dépôt :
```bash
git clone https://github.com/ralphi2811/tcp-proxy-docker.git
cd tcp-proxy-docker
```

2. Construisez l'image :
```bash
docker build -t tcp-proxy .
```

L'image est basée sur Alpine Linux 3.19.1 et utilise :
- bash : pour l'exécution du script
- socat : pour la redirection TCP

Pour utiliser votre image construite localement, remplacez `ralphi2811/tcp-proxy:latest` par `tcp-proxy` dans les exemples.

## Utilisation

Syntaxe de base :

```bash
docker run -d --restart always -p PORT_PUBLIC:PORT_CIBLE ralphi2811/tcp-proxy:latest IP_EQUIPEMENT PORT_CIBLE
```

### Configuration

| Variable d'environnement | Description | Valeur par défaut |
|-------------------------|-------------|-------------------|
| LISTEN_PORT | Port d'écoute du proxy | Même valeur que le port cible |

## Exemples d'utilisation

### 1. Exposition d'un TPE

Pour exposer un TPE situé à l'adresse 192.168.1.100 port 8080 sur le port public 9001 :

```bash
docker run -d --restart always -p 9001:8080 ralphi2811/tcp-proxy:latest 192.168.1.100 8080
```

### 2. Configuration multi-TPE avec Docker Compose

Voici un exemple de configuration `docker-compose.yml` pour gérer plusieurs redirections :

```yaml
version: '3.8'

services:
  # TPE du réseau A
  tpe1-proxy:
    image: ralphi2811/tcp-proxy:latest
    command: 192.168.1.100 8080
    ports:
      - "9001:8080"
    restart: always

  # TPE du réseau B
  tpe2-proxy:
    image: ralphi2811/tcp-proxy:latest
    command: 192.168.2.100 8080
    ports:
      - "9002:8080"
    restart: always

  # TPE du réseau C
  tpe3-proxy:
    image: ralphi2811/tcp-proxy:latest
    command: 192.168.3.100 8080
    ports:
      - "9003:8080"
    restart: always
```

Dans cet exemple :
- Chaque TPE est accessible via un port unique sur la VM publique
- Les conteneurs redémarrent automatiquement en cas de problème
- Chaque proxy peut cibler un TPE sur un réseau différent

## Port d'écoute

Par défaut, tcp-proxy écoutera sur le même port que celui spécifié.

Cette valeur peut être modifiée en définissant la variable d'environnement `LISTEN_PORT`.

## Sécurité

Points importants à considérer :
- N'exposez que les ports nécessaires
- Configurez un pare-feu (comme UFW ou iptables) sur la VM pour :
  - Restreindre l'accès aux ports exposés uniquement aux IP autorisées
  - Bloquer tout autre accès par défaut
  - Exemple avec UFW :
    ```bash
    # Bloquer tout le trafic par défaut
    ufw default deny incoming
    ufw default allow outgoing
    
    # Autoriser SSH (à adapter selon votre configuration)
    ufw allow from VOTRE_IP to any port 22
    
    # Autoriser les TPE uniquement depuis les IP spécifiques
    ufw allow from IP_AUTORISEE to any port 9001
    ufw allow from IP_AUTORISEE to any port 9002
    ufw allow from IP_AUTORISEE to any port 9003
    
    # Activer le pare-feu
    ufw enable
    ```
- Surveillez régulièrement les logs pour détecter toute activité suspecte :
  - Logs Docker : `docker logs nom_conteneur`
  - Logs système : `journalctl`
  - Logs UFW : `/var/log/ufw.log`

## Contribution

Les contributions sont les bienvenues ! N'hésitez pas à :

1. Fork le projet
2. Créer une branche pour votre fonctionnalité
3. Soumettre une Pull Request

## Licence

Ce projet est distribué sous licence MIT.
