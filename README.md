# Déploiement d'un SOC Cloud avec Wazuh (SIEM & EDR) sur AWS

Ce projet documente la mise en place d'une infrastructure de sécurité centralisée (SOC) utilisant la solution Wazuh. Déployé sur AWS, ce laboratoire simule un environnement de production hybride (Linux et Windows) pour tester les capacités de détection d'intrusions (SIEM) et de réponse aux incidents (EDR).

---

## Structure du Dépôt

L'arborescence du projet est organisée comme suit :

| Dossier | Description |
| :--- | :--- |
| **/Config** | Contient les fichiers ossec.conf (Agents et Manager) ainsi que les règles personnalisées XML. |
| **/Diagrams** | Schémas de l'architecture réseau et des flux de données (fichiers sources et PNG). |
| **/Assets** | Captures d'écran des preuves de concept (Alertes, Dashboard, Logs). |
| README.md | Documentation technique du projet. |

---

## Architecture Technique

L'infrastructure repose sur une topologie Hub-and-Spoke au sein d'un VPC AWS sécurisé.

![Schéma SOC Cloud Wazuh](./Assets/schema_Aws_Wazuh.drawio.png)
*(Enregistrez le schéma fourni sous le nom aws_soc_architecture.png dans le dossier Diagrams.)*

### Composants

1. **Wazuh Manager (SIEM/Indexer/Dashboard)** : instance EC2 t3.large (Ubuntu). Cœur du système qui agrège et analyse les logs.
2. **Linux Agent** : instance EC2 t2.micro (Ubuntu). Cible pour les attaques SSH et l'élévation de privilèges.
3. **Windows Agent** : instance EC2 t2.medium (Windows Server). Cible pour les attaques RDP et la persistance (Backdoor).

### Flux Réseau et Sécurité (Security Groups)

| Protocole | Port | Source | Destination | Usage |
| :--- | :--- | :--- | :--- | :--- |
| TCP | 1514 | Security Group des Agents | Wazuh Server | Remontée des logs via transport sécurisé. |
| TCP | 1515 | Security Group des Agents | Wazuh Server | Enrôlement automatique des agents. |
| TCP | 443 | Adresse IP d'administration | Wazuh Dashboard | Accès HTTPS à l'interface d'administration. |
| TCP | 22 / 3389 | Adresse IP d'administration | Instances | Administration SSH (Linux) et RDP (Windows). |

---

## Installation et Configuration

### 1. Déploiement du Manager

Installation « All-in-one » sur Ubuntu :

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

### 2. Configuration des Agents (dossier Config)

Les fichiers de configuration des agents (ossec.conf) pointent vers l'adresse IP privée du Manager afin de rester dans le réseau local AWS.

Exemple d'extrait (Config/ossec_linux.xml) :

```xml
<client>
	<server>
		<address>172.31.29.180</address>
		<port>1514</port>
		<protocol>tcp</protocol>
	</server>
</client>
```

---

## Scénarios d'Attaque et Détection (POC)

### A. Vecteur Linux : Authentification et Privilèges

- Attaque : simulation d'un brute force SSH via Hydra ou tentatives manuelles, suivie d'une élévation de privilèges (sudo su).
- Détection Wazuh :
	- Rule ID 5710 : sshd, tentative de connexion avec un utilisateur inexistant.
	- Rule ID 5501 : PAM, ouverture d'une session root.

### B. Vecteur Windows : Persistance et Intégrité

- Attaque : création d'un utilisateur « backdoor » et ajout au groupe Administrateurs via PowerShell.

```powershell
net user labuser P@ssw0rd! /add
net localgroup administrators labuser /add
```

- Détection Wazuh :
	- Rule ID 60170 : création d'utilisateur (surveillance des comptes locaux).
	- Rule ID 60154 : modification du groupe Administrators (détection de persistance critique).

---

## Perspectives d'Amélioration

- Intégrer Sysmon sur Windows pour bénéficier d'une granularité supplémentaire sur la création de processus (Event ID 1).
- Mettre en place Wazuh Active Response pour bannir automatiquement les adresses IP attaquantes.

---

## Auteur

[Votre Nom]

Étudiant ingénieur en Génie Logiciel – ENSET Mohammedia

Projet réalisé dans le cadre du module « Sécurité des Endpoints et SIEM ».

---

## Instructions pour Publier

1. Créez un fichier nommé README.md à la racine de votre dossier projet sur votre ordinateur.
2. Collez le contenu ci-dessus dedans.
3. Remplacez [Votre Nom] par votre vrai nom à la fin.
4. Vérifiez que vos images se trouvent dans les dossiers correspondants (Diagrams et Assets) pour que les liens fonctionnent.
5. Exécutez git push.

